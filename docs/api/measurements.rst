.. _api_measurements:

Using the measurement API
=========================

Measurements are how wger stores body metrics: body weight, body fat,
circumferences, blood pressure, heart rate, sleep and anything else a user
wants to keep track of. This page explains the model; for the request and
response schemas of every endpoint, see ``/api/v2/schema/redoc``.

Object model
------------

There are two objects, each with its own endpoint:

.. code-block:: text

   Category           /api/v2/measurement-category/
   └── Measurement    /api/v2/measurement/

A **category** is one thing that gets tracked, together with the unit it is
tracked in ("Biceps" in cm, "body weight" in kg). A **measurement** is a
single value in such a category, with a timestamp.

Categories can be nested exactly one level deep through their ``parent``
field. This is how multi-value metrics are modelled: the group is itself a
category and holds one child category per component. Only the leaves carry
measurements:

.. code-block:: text

   Blood pressure     metric_type=blood_pressure          (group)
   ├── Systolic       metric_type=blood_pressure_systolic (component)
   └── Diastolic      metric_type=blood_pressure_diastolic

Deleting a category deletes its children and all their measurements.


Measurements
------------

A measurement has ``category``, ``date``, ``value``, ``notes``, plus the
three fields the sync uses (``source``, ``external_id``, ``extra_data``).

``date`` is a timezone-aware datetime and there is no one-entry-per-day
restriction: a reading in the morning and one in the evening are two entries,
and the components of one reading share their exact timestamp.

The list is ordered by ``-date, -id``. The id is part of it because entries
written by the health sync share a timestamp, and paging needs a total order
to not skip or repeat rows.


Extra data
~~~~~~~~~~

``extra_data`` is a free JSON object per entry, limited to 1000 bytes, with
three reserved keys.

``unit`` holds the unit the value was entered in, so one category can hold
mixed units: body weight in kilograms at home and in pounds on vacation.
Without it the unit of the category applies, which is why a body weight entry
that arrives without one is stamped with it. Body weight is also the only type
where the server checks the unit, ``kg`` or ``lb`` and nothing else; elsewhere
it is a label that nothing converts.

``min`` and ``max`` hold the range an entry that summarises a whole day covers,
as numbers rather than numeric strings: the chart endpoints below read them as
such. The health importer writes its provenance into the same object.

.. note::
    ``extra_data`` is replaced as a whole, also on ``PATCH``. Read the
    object, merge your keys into it and send it back, otherwise you drop
    the keys somebody else put there.


Value limits
~~~~~~~~~~~~

Values are validated against the range their metric type allows. The bounds
answer "beyond this it is almost certainly a typo":

.. list-table::
   :header-rows: 1
   :widths: 45 15 40

   * - metric_type
     - Unit
     - Accepted range
   * - ``body_weight``
     - kg
     - 20 to 350
   * - ``body_weight``
     - lb
     - 44 to 770
   * - ``body_fat``
     - %
     - 2 to 60
   * - ``lean_body_mass``
     - kg
     - 10 to 250
   * - ``height``
     - cm
     - 50 to 250
   * - ``blood_pressure_systolic``
     - mmHg
     - 50 to 250
   * - ``blood_pressure_diastolic``
     - mmHg
     - 30 to 150
   * - ``heart_rate``
     - bpm
     - 30 to 250
   * - ``resting_heart_rate``
     - bpm
     - 30 to 120
   * - ``blood_oxygen``
     - % per day
     - 50 to 100
   * - ``steps``
     - count per day
     - 0 to 100 000
   * - ``distance``
     - km per day
     - 0 to 500
   * - ``energy``
     - kcal per day
     - 0 to 10 000
   * - ``sleep_total``, ``sleep_light``, ``sleep_deep``, ``sleep_rem``,
       ``sleep_awake``
     - minutes per day
     - 0 to 1440
   * - ``custom``
     - free
     - 0 to 999999.99

Body weight is the only type whose bounds depend on the unit. For every other
type the unit above is a convention the clients follow; a category carrying
something else still gets the bounds of its metric type.

The clients mirror this table to validate while offline, so released bounds
may only ever be widened, never tightened: an older client with wider bounds
would keep pushing values the server rejects permanently.


Imported and calculated entries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Three fields describe where an entry came from:

``source``
    ``user`` (the default, entered by hand), ``google`` (Health Connect),
    ``apple`` (Apple Health) or ``calculated`` (written by the server for a
    calculated category).

``external_id``
    The record id in the source platform, as a UUID. A calculated entry uses
    it to point at what it was derived from, either the id of the entry it
    was computed from or an id derived from the day it summarises.

``extra_data``
    The provenance of the import, see above.

``category``, ``source`` and ``external_id`` together are unique, which makes
a re-import idempotent: an entry the importer already wrote is refused with a
``400`` instead of duplicated, and the server can recompute a calculated
category the same way. ``external_id`` cannot be used as a filter.

The mobile app shows entries with a ``source`` other than ``user`` as
read-only, which for the imported ones is a client convention: the importer
has to be able to write its own rows. For ``calculated`` the API enforces it,
since anything written there is replaced by the next recomputation anyway:

* posting an entry into a calculated category is a ``400``
* changing a calculated entry is a ``400``, deleting one a ``403``
* entries the user wrote themselves stay editable, the rule follows the
  entry rather than the category

A category is recomputed whenever something it derives from changes, plus
once a day as a safety net, so the new values may arrive a moment after the
change that caused them.

.. note::
    Clients should treat ``source`` as an open list. A value they do not
    know means "not written by the user here", which is the only thing the
    UI needs from it. New values are added without a major version, so a
    strict mapping is a source of hard errors later.


Categories
----------

Every measurement belongs to a category, and what the category is decides
what may be written into it: the unit, the bounds of the values, and whether
the user writes them at all.


Metric types
~~~~~~~~~~~~

``metric_type`` says what a category measures, and everything else hangs off
it: the unit, the value limits, the chart the clients draw and the mapping to
Apple Health and Health Connect. The default is ``custom``, a free-form
category with no semantics attached.

Each type has exactly one role:

.. list-table::
   :header-rows: 1
   :widths: 15 45 40

   * - Role
     - Types
     - Rule
   * - leaf
     - ``body_weight``, ``body_fat``, ``lean_body_mass``, ``height``,
       ``heart_rate``, ``resting_heart_rate``, ``blood_oxygen``, ``steps``,
       ``distance``, ``energy``
     - Top level, carries the measurements, never has children
   * - group
     - ``blood_pressure``, ``sleep``
     - Top level, container only, carries no measurements at all
   * - component
     - ``blood_pressure_systolic``, ``blood_pressure_diastolic``,
       ``sleep_total``, ``sleep_light``, ``sleep_deep``, ``sleep_rem``,
       ``sleep_awake``
     - Only as a child, and only under its own group

These rules are enforced by the API, a violation is a ``400``:

* a component category needs a ``parent`` of the matching group type
* any other typed category cannot be nested
* a group only accepts its own components as children
* measurements cannot be posted to a group category, nor to any category
  that has children
* a category that already has measurements cannot be used as a parent

Creating a group category also creates its components: a ``POST`` with
``metric_type=blood_pressure`` gives you the group plus its "Systolic" and
"Diastolic" children in one request.

The type is set once, sending a different one for an existing category is a
``400``. It is not an attribute but the category's identity: its primary key
is derived from the type (see the client-assigned ids below). Giving a type to
a category you already have means creating the typed one and moving the
entries over.

``sleep_total`` is a component like the others, not a computed sum: the
importer fills it with the whole night, including the parts a platform reports
without a sleep stage.


One category per metric type
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A user has at most one category per metric type, enforced by a database
constraint (``custom`` is exempt); a second one is a ``400``. A typed category
is the one place its metric lives, so manual and imported entries end up
together and the mobile app can look its target up by type.


Official categories
~~~~~~~~~~~~~~~~~~~

``is_official`` marks a category the server itself depends on. Today that is
the body weight category, created automatically for every account and backing
the ``weightentry`` endpoint described below.

The field is read-only and filterable. An official category cannot be deleted
(``403``) and keeps its ``metric_type``; everything else about it, the name
and the chart included, is up to the user.


Calculated categories
~~~~~~~~~~~~~~~~~~~~~

A category can have its entries computed by the server instead of entered by
the user: ``dynamic_type`` selects what is computed, ``dynamic_params``
configures it.

The result is **stored as ordinary measurements**, not computed on read, so
they are filtered, paged, charted and synced like every other entry and no
client has to implement a formula. Their ``source`` is ``calculated``.

The available types come from the server, with the schema their parameters
have to match::

    GET /api/v2/measurement-category/dynamic-types/

    [{"value": "BMI", "label": "BMI", "params_schema": {...}}, ...]

.. list-table::
   :header-rows: 1
   :widths: 20 30 50

   * - ``dynamic_type``
     - ``dynamic_params``
     - What is computed
   * - ``BMI``
     - none
     - One entry per body weight entry, from the official body weight
       category and the height in the user profile
   * - ``WHTR``
     - ``category_id``
     - Waist to height ratio: one entry per entry of the given category,
       divided by the height in the user profile
   * - ``ONE_REP_MAX``
     - ``exercise_id``, ``max_reps``
     - One entry per day the exercise was trained, holding the highest
       one-rep-max estimate of that day
   * - ``ONE_RM_TOTAL``
     - ``exercise_ids``, ``max_reps``, ``window_days``
     - The sum of several exercises, one entry per day one of them was
       trained, see below

Both one-rep-max types estimate with the Brzycki formula over sets of at most
``max_reps`` repetitions (5 by default, 10 at the most), since it degrades at
higher counts. Sets in pounds are converted, sets counted in something other
than repetitions are ignored. A day is a calendar day in the timezone of the
user profile, and only a day with a qualifying set gets an entry.

``ONE_RM_TOTAL`` takes two to five exercises and adds up the best estimate per
exercise within the last ``window_days`` days (30 by default, 7 to 120), so
the total tracks what can be lifted now rather than a personal best that never
expires. A day only gets an entry when every exercise has a qualifying set in
its window, since a partial sum would read as a drop.

Which categories can be calculated:

* only ``custom`` ones. A typed category already has a writer, the health
  import or the server itself, and two of them would fight over the same rows.
* only ones that hold no entries of their own, move or delete those first.
* only once per configuration. A second category with the same
  ``dynamic_type`` and ``dynamic_params`` is a ``400``, it would compute the
  same series twice. The same calculation with other parameters is fine.

``dynamic_params`` is validated against the schema of the type and against the
data behind it: the source category of a ``WHTR`` has to exist, belong to the
user, not be calculated itself and be measured in a length (mm, cm, m or
inches), and the exercises of the one-rep-max types have to exist. Only a
request that changes ``dynamic_type`` or ``dynamic_params`` is checked, so
sending the category back unchanged never fails on data that has since been
deleted.

``dynamic_type`` is set once, like ``metric_type``: sending a different one,
``NONE`` included, is a ``400``. To stop calculating, delete the category; its
computed entries go with it and the data they were derived from stays. The
parameters stay editable, the next reconciliation rewrites the series
anyway.


Chart, order and client settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Three fields on a category exist for the clients rather than for the data:

``chart_type``
    ``line``, ``bar``, ``heatmap``, ``delta`` or ``distribution``, and
    ``null`` by default, which means "the chart the metric type implies".
    Which values a client offers, and what it falls back to for one that
    does not fit the category, is up to the client.

``chart_config``
    A free JSON object for the settings that are a matter of taste, e.g.
    ``{"trend": "sluggish"}``. The keys are the clients' business and a
    missing one means their default; the server only checks that it is an
    object of at most 1000 bytes. Like ``extra_data`` above it is replaced as
    a whole, also on ``PATCH``.

``order``
    The position in the category list, and for a component the position
    inside its group. Categories come back ordered by ``order``, then name.


Filtering and ordering
----------------------

Categories can be filtered by ``id``, ``name``, ``unit``, ``metric_type``,
``parent`` and ``is_official``.

Measurements can be filtered by ``id`` (also ``id__in``), ``category``
(also ``category__in``), ``source`` and ``date``, the latter with the
``date__gt``, ``date__gte``, ``date__lt`` and ``date__lte`` variants. For
charts, fetch the range you draw instead of the whole history, the endpoint
is indexed for exactly that access pattern.

Both endpoints support ``?ordering=`` and the usual pagination, see
:doc:`api`.


Reading a chart
---------------

A history the health sync writes to can be tens of thousands of rows, so two
read-only endpoints condense them server side. Both take the filters of the
list endpoint, so one call covers a whole group via ``category__in``, and both
group by the unit an entry was stored in.

``GET /api/v2/measurement/aggregate/``
    One row per category, calendar bucket and unit:

    .. code-block:: json

        [{"category": "...", "start": "2026-08-01T00:00:00+02:00",
          "unit": "kg", "count": 3, "sum": "241.50",
          "min": "80.10", "max": "80.90"}]

    ``bucket`` is ``auto`` (the default), ``hour``, ``day``, ``week`` or
    ``month``, where ``auto`` picks the finest unit that keeps the series
    under ``max_points`` (200 by default). ``tz`` takes the IANA name of the
    zone the buckets are cut in and defaults to the server's: the column is
    UTC, and a reading taken after midnight belongs to the day the user had
    it.

``GET /api/v2/measurement/value-counts/``
    How often each value occurred and when it was measured last, which is
    what a histogram bins:

    .. code-block:: json

        [{"category": "...", "value": "80.50", "unit": "kg",
          "count": 4, "newest": "2026-08-09T07:12:00+02:00"}]

    Takes ``tz`` as well, plus ``summed_per_day`` for the metrics whose
    single samples say nothing on their own (steps, sleep), which are then
    counted as daily totals instead of as readings.

A bucket unit or a timezone that does not exist is a ``400``.


Client-assigned ids
-------------------

Both objects use UUIDs as primary key, and ``id`` is writable. Most clients
never need that and simply leave it out. Sending your own is what lets an
offline client create rows and push them later without duplicating them on a
re-sync: an id that already exists is answered with a ``400`` instead of with
a second row.

Typed categories are the exception: their id follows from user and metric
type, and the server sets it whether the request brought one or not, so two
devices creating the same category offline arrive at the same row.
``body_weight`` is created by the server itself, once per account, so a
``POST`` for another one is refused.


The weightentry endpoint
------------------------

``/api/v2/weightentry/`` still exists, as a compatibility endpoint for
integrations written against it. Do not build on it: it is the measurements
of the official body weight category behind a narrower shape, its values are
always in the unit of the user profile, and it will go away.

Use ``/api/v2/measurement/`` with the official body weight category instead
(find it with
``/api/v2/measurement-category/?metric_type=body_weight&is_official=True``).
