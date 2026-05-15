.. _api_routines:

Using the routine API
=====================

The routine data model is split across a handful of related objects and
has a few non-obvious moving parts (iterations, day sequencing,
progression rules). This page explains how everything fits together. It
is **not** a field reference: for the full request/response schemas of
every endpoint, use ``/api/v2/schema/redoc/``.

A routine has a maximum duration of 120 days.


Object hierarchy
----------------

The routine itself is just a shell. The actual training data is split
across several linked objects, each with its own endpoint:

.. code-block:: text

   Routine                           /api/v2/routine/
   └── Day                           /api/v2/day/
       └── Slot                      /api/v2/slot/
           └── SlotEntry             /api/v2/slot-entry/
               ├── WeightConfig      /api/v2/weight-config/      (+ max-)
               ├── RepetitionsConfig /api/v2/repetitions-config/ (+ max-)
               ├── SetsConfig        /api/v2/sets-config/        (+ max-)
               ├── RirConfig         /api/v2/rir-config/         (+ max-)
               └── RestConfig        /api/v2/rest-config/        (+ max-)

The actual training history lives in two more objects, both linked back
to the routine:

.. code-block:: text

   WorkoutSession                    /api/v2/workoutsession/
   └── WorkoutLog                    /api/v2/workoutlog/

Templates are exposed read-only through ``/api/v2/templates/`` (your
own) and ``/api/v2/public-templates/`` (those shared by others).


Iterations
----------

An "iteration" is one complete cycle through all the days of a routine.
Once the last day has been done, the next occurrence of the first day
starts a new iteration. In practice this will most often be a week, but
the system doesn't require that.

The iteration number is the key that ties progression rules to specific
points in the routine: a ``WeightConfig`` with ``iteration=3`` and
``value=60`` applies starting on the third pass through the days.


Days and day sequencing
-----------------------

A routine is built from an ordered list of days (controlled by each
day's ``order`` field). From this list, plus the routine's start and
end date, wger computes a concrete date-by-date sequence.

For a routine that starts on the 1.1 with three days, the basic
sequence is:

.. list-table::
   :header-rows: 1

   * -
     - 1.1
     - 1.2
     - 1.3
     - 1.4
     - 1.5
     - 1.6
     - 1.7
     - 1.8
     - 1.9
     - 1.10
   * - Day
     - Day 1
     - Day 2
     - Day 3
     - Day 1
     - Day 2
     - Day 3
     - Day 1
     - Day 2
     - Day 3
     - Day 1
   * - Iteration
     - 1
     - 1
     - 1
     - 2
     - 2
     - 2
     - 3
     - 3
     - 3
     - 4

Two flags change this default behaviour:

**need_logs_to_advance** (on the day) stalls the sequence on a day
until the user logs a session for it. Below, Day 3 has the flag set,
the user finally logged a session on 1.8, and the sequence resumes on
1.9:

.. list-table::
   :header-rows: 1

   * -
     - 1.1
     - 1.2
     - 1.3
     - 1.4
     - 1.5
     - 1.6
     - 1.7
     - 1.8
     - 1.9
     - 1.10
   * - Day
     - Day 1
     - Day 2
     - Day 3
     - Day 1
     - Day 2
     - Day 3
     - Day 3
     - Day 3
     - Day 1
     - Day 2
   * - Iteration
     - 1
     - 1
     - 1
     - 2
     - 2
     - 2
     - 2
     - 2
     - 3
     - 3

**fit_in_week** (on the routine) pads each iteration with empty
placeholder days (no day, no exercises) so the routine always restarts
on a fixed weekday:

.. list-table::
   :header-rows: 1

   * -
     - 1.1 (Mon)
     - 1.2
     - 1.3
     - 1.4
     - 1.5
     - 1.6
     - 1.7
     - 1.8 (Mon)
     - 1.9
     - 1.10
   * - Day
     - Day 1
     - Day 2
     - Day 3
     - --
     - --
     - --
     - --
     - Day 1
     - Day 2
     - Day 3
   * - Iteration
     - 1
     - 1
     - 1
     - 1
     - 1
     - 1
     - 1
     - 2
     - 2
     - 2

Rest days are regular days with ``is_rest=true`` and no slots attached.
They occupy a position in the sequence but produce no exercises.


Slots and slot entries
----------------------

Inside a day, exercises aren't attached directly. They go through a
two-level structure:

* a **slot** is one "position" in the day: roughly one exercise (or
  one superset)
* a **slot entry** attaches a specific exercise to that slot

If a slot has more than one entry, it automatically becomes a superset.
The ``/date-sequence-gym`` view interleaves the sets across the
entries; they don't need to have the same number of sets. With

* Exercise 1, 4 sets
* Exercise 2, 2 sets
* Exercise 3, 3 sets

the gym-mode output is:

.. code-block:: text

   Exercise 1
   Exercise 2
   Exercise 3
   Exercise 1
   Exercise 2
   Exercise 3
   Exercise 1
   Exercise 3
   Exercise 1

The ``repetition_rounding`` and ``weight_rounding`` fields on a slot
entry control how the computed values are rounded for display. If left
empty when creating the entry, the defaults from the user profile are
copied in.


Progression rules
-----------------

The actual values for weight, repetitions, sets, RiR and rest aren't
stored on the slot entry. They live in separate config objects (one
endpoint per field), each with a ``max-`` variant for ranges like
"8 to 10 reps".

Each config record applies starting at a given ``iteration``. The value
at iteration *n* is the stacked result of all preceding records: each
record either replaces the value outright (``operation=r``) or adds /
subtracts a value (``+`` / ``-``), either as an absolute number
(``step=abs``) or a percentage (``step=percent``). The ``repeat`` flag
keeps a rule active for every following iteration until another rule
takes over, so "+1 kg every week" is a single record.

A weight config example:

.. list-table::
   :header-rows: 0

   * - **Iteration**
     - 1
     - 2
     - 3
     - 4
     - 5
     - 6
     - 7
     - 8
   * - **Config**
     - 50kg
     - --
     - --
     - +10%
     - --
     - +2kg
     - +1kg
     - 45kg
   * - **Result**
     - 50kg
     - 50kg
     - 50kg
     - 55kg
     - 55kg
     - 57kg
     - 58kg
     - 45kg

All configs are optional. If none is set for a field, that field will
be ``null`` in the computed output, except for sets, which defaults to
1.


Requirements
~~~~~~~~~~~~

A progression rule can be made conditional on the user actually hitting
the prescribed values in the previous iteration. Set the
``requirements`` field to a JSON object of the form::

    {
        "rules": [
            "weight",
            "repetitions",
            "rir",
            "rest"
        ]
    }

If the ``rules`` list is non-empty, the rule only applies when *all*
listed fields were met in at least one log of the previous iteration.
For example: if the target is to move from 8x60 kg to 8x65 kg, with
``rules`` containing ``weight`` and ``repetitions``, the increase only
happens once the user has actually logged 8 reps at 60 kg. Until then
the field stays at the previous value.


Custom calculation logic
~~~~~~~~~~~~~~~~~~~~~~~~

If the rule-based system is not flexible enough, a slot entry's
``class_name`` field can point to a Python class under
``wger.manager.config_calculations`` that takes over all calculations
for that entry. This is an escape hatch and not currently used; please
get in touch if you have a use case for it.


Sessions and logs
-----------------

The training history is stored in two objects. A **WorkoutSession**
represents one workout (one date, optional notes and impression). A
**WorkoutLog** is one performed set, attached to a session and
optionally back-linked to a routine, day, slot entry and iteration.

Each log carries both what was actually performed (``weight``,
``repetitions``, ``rir``, ``rest``) and the originally prescribed
values (``weight_target``, ``repetitions_target``, ``rir_target``,
``rest_target``), so a log preserves both sides even when the routine
later changes.


Computed endpoints
------------------

Once a routine is set up, these read-only sub-resources return
calculated views of it:

``/api/v2/routine/{id}/structure/``
    The full nested structure (routine → days → slots → slot entries),
    suitable for an editor.

``/api/v2/routine/{id}/date-sequence-display/``
    One entry per date, with the matching day and slots already
    resolved. Slots that contain repeated sets of the same exercise
    are folded together for display.

``/api/v2/routine/{id}/date-sequence-gym/``
    Same data, but the slots are split into individual sets and
    supersets are interleaved as described above. Use this for the
    gym-mode view.

``/api/v2/routine/{id}/logs/``
    The workout sessions and logs for this routine, grouped by
    session.

``/api/v2/routine/{id}/stats/``
    Aggregated statistics from the logs: volume, set count and
    average intensity (estimated 1RM via the Brzycki formula), each
    broken down by **day**, **ISO week**, **iteration** and the
    **whole routine**, and further split into total, upper/lower body,
    per muscle and per exercise.

All five endpoints are cached server-side and invalidated when the
underlying routine changes.
