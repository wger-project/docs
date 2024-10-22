.. _routines:

Using the routine API
=====================

This section explains how the data model behind the routines and its different
calculations work and is only relevant to you if you plan on using the API
directly.

A routine has

* ``name``, 50 chars max
* ``description``, optional, 1000 chars max
* a ``start date``
* an ``end date``

Days
----

A day is the building block of a routine. It consists of of several linked days
that can be configured in different ways.

::

   +----------------+      +-----------------+      +-----------------+
   |Day 1 - Push    |      |Day 2 - Pull     |      |Day 3 - Legs     |
   |----------------+      |-----------------+      |-----------------+
   |order=1         | ---> |order=2          | ---> |order=3          |
   |rest=false      |      |rest=false       |      |rest=false       |
   |needs_logs=true |      |needs_logs=false |      |needs_logs=false |
   +----------------+      +-----------------+      +-----------------+

The routine will calculate from the list a sequence of days that are planned.
This allows you to create very flexible setups. If the routine starts on the 1.1, it
will create the following sequence:

.. list-table::
   :header-rows: 1

   * - 1.1
     - 1.2
     - 1.3
     - 1.4
     - 1.5
     - 1.6
     - 1.7
     - 1.8
     - 1.9
     - 1.10
   * - Day 1
     - Day 2
     - Day 3
     - Day 1
     - Day 2
     - Day 3
     - Day 1
     - Day 2
     - Day 3
     - Day 1
   * - 1
     - 1
     - 1
     - 2
     - 2
     - 2
     - 3
     - 3
     - 3
     - 4


With the ``need_logs_to_advance`` flag you can control whether there needs to be a
logged session for the day for the day to proceed. Otherwise, the day will be repeated
till a log is saved, like in the example below with day 3 where the user logged a
session on the 1.8.

.. list-table::
   :header-rows: 1

   * - 1.1
     - 1.2
     - 1.3
     - 1.4
     - 1.5
     - 1.6
     - 1.7
     - 1.8
     - 1.9
     - 1.10
   * - Day 1
     - Day 2
     - Day 3
     - Day 1
     - Day 2
     - Day 3
     - Day 3
     - Day 3
     - Day 1
     - Day 2
   * - 1
     - 1
     - 1
     - 2
     - 2
     - 2
     - 2
     - 2
     - 3
     - 3

Days marked as "rest" can be used to pad training days and do not contain any exercises.

The bottom row shows the *iteration* of the day, which is the number of times that day
has occurred and is used to calculate the current weight, reps, etc. If a day needs a log
to advance and it repeats, the iteration counter will not increase.

You can also set the ``last_day_in_week`` flag to indicate that this is the last workout
day in the current week. The API will return null for days and iterations for the remainder
of the week.


Here day 3 has the flag set:

.. list-table::
   :header-rows: 1

   * - 1.1 - Monday
     - 1.2
     - 1.3
     - 1.4
     - 1.5
     - 1.6
     - 1.7
     - 1.8 - Monday
     - 1.9
     - 1.10
   * - Day 1
     - Day 2
     - Day 3
     - null
     - null
     - null
     - null
     - Day 1
     - Day 2
     - Day 3
   * - 1
     - 1
     - 1
     - null
     - null
     - null
     - null
     - 2
     - 2
     - 2

Additionally days have the following properties:

* ``type`` the type of workout day. These will change the way the workout for a
  specific day is handled, but currently this setting is ignored. Current values are:
   * ``custom`` (default)
   * ``enom``
   * ``amrap``
   * ``hiit``
   * ``tabata``
   * ``edt``
   * ``rft``
   * ``afap``
* ``name``, 50 chars max
* ``description``, optional, 1000 chars max
* ``is_rest`` flag indicating that this is a rest day

Labels
``````
You can group the workout days (such as "Deload week" or similar) by adding labels to
the routine. A label accepts the following properties:

* ``start_offset``, the number of days after the start of the routine when this label becomes active
* ``end_offset``, the number of days after the start of the routine when this label ceases being active
* ``label``, the label that will be displayed in the UI



Sets
----

You can add exercises to a slot (set). These slots have a ``SlotConfig``
entry, and different individual config entries for individual properties where
the magic happens.

* ``repetition_rounding`` - the rounding factor for the repetition value (happens client side)
* ``weight_rounding`` - the rounding factor for the weight value (happens client side)
* ``type`` the type of set. These will change the way the set is handled or displayed,
  but currently this setting is ignored. Current values are:
   * ``normal`` (default)
   * ``dropset``
   * ``myo``
   * ``partial``
   * ``forced``
   * ``tut``
   * ``iso``
   * ``jump``


Supersets
`````````

If you add more than one exercise to a set, it automatically becomes a superset.
The specific oder of exercises is the interleaved list of exercises. Not all exercises
need to have the same number of sets, e.g.:

* Exercise 1, 4 sets
* Exercise 2, 2 sets
* Exercise 3, 3 sets

Would result in:

* Exercise 1
* Exercise 2
* Exercise 3
* Exercise 1
* Exercise 2
* Exercise 3
* Exercise 1
* Exercise 3
* Exercise 1
(with the respective values for weight, reps, etc.)


Configuration
`````````````

There are config tables / endpoints for the following properties:

* nr of sets
* weight
* repetitions
* reps in reserve (RiR)
* rest time

All of these are optional, in which case they will return null over the API.
In this case the number of sets will be set to 1.

The behaviour is basically the same for all of them, here with a weight config example:

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
     - -/-
     - -/-
     - +10%
     - -/-
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

You can add changes that will happen at specific iterations and either modify the
weight (+2kg, -10%) or replace it with a new value (45). The value at a specific iteration
is the stacked calculated value (unless you just replace the value with a new one) of
the previous ones. There are also a handful of possibilities on how to calculate the value
such as increasing / decreasing or using an absolute value or a percentage.

When exactly an iteration happens depends on how the days are configured and
whether logs are required from the user or not.

One of the ways the configs currently differ is the handling of the ``need_log_to_apply``
flag. If this is set for both the weight and reps value, the system will check that
the user logged at least the planned weight and reps. As an example, if your weight
should change from 8x60 to 8x65 but you didn't log at least that in the last workout,
you will stay at 8x60 till you do. For all other fields this flag is currently
ignored.

If this is not enough, there is an escape hatch in the form of setting a custom python
class that can perform any calculations you might need. Please consider that while this
works, it is not currently in use so we would be happy if you got in touch with us.

Possible values:
~~~~~~~~~~~~~~~

* ``value``: Decimal number with the wanted value
* ``operation``: Operation to perform: ``+``, ``-`` for adding or subtracting the value, or to replace it ``r``
* ``step``: How to calculate the new value: ``abs`` or ``percent``
* ``need_log_to_apply``: Boolean, whether a valid log is required to proceed

