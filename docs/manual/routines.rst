.. _manual_routines:

Using the routines
==================

A routine is a planned sequence of workouts. It automatically tells you which
workout you have to do next, so for example, if you did an "arms" workout today,
a routine can remind you that "chest and shoulders" is next.

.. note::
    The routines are made around the assumption that you will do the same workout
    for some weeks. If you prefer to do ad-hoc workouts with whatever exercises
    you feel, you will not be happy with this app!


Workout days
------------

The basic building block of a routine is a **day**. A "day" can either be a
regular workout day with a list of exercises, or it can be a designated **rest
day**.

You have two main options for setting up the timing and sequence of the workout
(in either case, it's probably better to make sure the first day of the routine
is a Monday, but it's not necessary):


**Fit the routine in a week**

If you want a consistent weekly schedule, like working out every Monday,
Wednesday, and Friday or just "three times a week" without any specific weekdays,
simply add your workout days (e.g., three different workout days) and enable the
``fit in week`` toggle. Your routine will then restart every week on the same day.

If using this mode, it's probably a good idea to activate the "needs logs to
advance" toggle (see next section).

**Create a custom cycle**

This option gives you maximum flexibility to create routines with unique lengths
and periodizations. You can explicitly add all your workout and rest days in
the exact order you want them to repeat, even if they don't fit into one week.

For example a cycle of ``Day A → Day B → Rest → Rest → Day C → Rest``
will repeat in that exact sequence, repeating every 6 days.


Advancing through the routine
------------------------------

By default, the routine advances to the next day automatically. However, you can
control this behavior.

If you want to ensure you complete a workout before the routine moves on, you can
enable the ``needs logs to advance`` toggle for any workout day. When this is
active, that day remains the active one until you log at least one set for any of
its exercises. This is useful for making sure you don't miss any sessions.

Note that this toggle does not need to be applied to all days. For example: you
have a routine with a "Monday group class" workout day, but for the others you go
alone. If you can't attend one week, leaving the toggle off for that day allows the
routine to advance automatically. This keeps your schedule on track without
requiring you to log a session you didn't do.

The currently active day in the routine is then marked with a calendar icon on
the dashboard.


Sets and exercises
------------------

A workout day is a collection of exercises you plan to do in a single session.
You can add as many sets as you need. By default, the interface is kept simple,
if you need more control, you can use the ``simple mode`` toggle. This will
reveal more options, allowing you to:

* Set other units (e.g., minutes, distance).
* Specify an explicit rest time between sets.
* Specify a min and max value for the settings


**Supersets**

If you add more than one exercise to a set, it automatically becomes a superset.
When you run the workout in gym mode, exercises are presented in an alternating
sequence. Also note that not all exercises need to have the same number of sets,
e.g.:

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

(with their respective values for weight, reps, etc.)


Progression rules
-----------------

.. note::
    This feature is currently only available on the **web version**. However, the
    resulting routines can be used and viewed on the mobile app as well.

For more advanced control over your progress, you can set rules for automatic
changes in weight, repetitions, number of sets or RiR. You have the option to

* add or substract a fixed value
* add or substract a percentage (in this case it's probably a good idea
  to also set a rounding value)
* reset the value to a specified number

in any future week.

You can also set the ``repeat rule`` toggle so that the rule will be repeated
until a new rule is encountered.

In case you don't want these values to change automatically, you can also configure
their ``requirements``, which are rules that make it so that a planned progression
only takes effect if certain conditions are met.

For example you can create a rule to increase the weight of an exercise by 1.25kg
every week. You can then set a requirement that this increase only happens if you
successfully completed all the planned reps and weight from a previous session.
If you didn't meet the requirement, the weight will stay the same until you do.
