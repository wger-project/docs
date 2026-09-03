.. _manual_measurements:

Measurements and health data
============================

Besides your workouts, wger keeps track of body data: your weight, your body
fat, the circumference of your arms, your blood pressure, your sleep, and
anything else you want to measure yourself. Since version 2.7 the mobile app
can also import part of that automatically from Apple Health or Health
Connect.


Categories
----------

Everything you measure lives in a **category**, which is simply a name and a
unit: "Biceps" in cm, "Waist" in cm. You add entries with a date and a value,
the app draws a chart from them, and you can drag the categories into the
order you want to see them in.

Some categories know **what** they measure: body weight, body fat, blood
pressure, heart rate, sleep, steps and a few more. Such a category gets its
unit, its chart and its plausibility check from that, and it is where the
health sync writes. You can have only one per type, so there is exactly one
place where your blood pressure lives. Everything else is a free-form
category where you pick the name and the unit yourself.

Measurements made of more than one number are stored as a **group** with one
sub-category per part: blood pressure has a systolic and a diastolic one,
sleep the total plus its phases. The parts are shown together, so a reading
reads as "120/80" and a night as one bar made of its phases.


Body weight
-----------

Your body weight is a category like the others, but it gets its own screen,
it feeds the BMI and the calorie calculations, and it is created
automatically for every account, so it cannot be deleted.

Every entry remembers the unit it was entered in, so you can log in kilograms
at home and in pounds on vacation. Everything is shown in the unit set in
your profile; changing that preference does not reinterpret your history,
only how it is displayed.


Categories that calculate themselves
------------------------------------

Some things follow from what you already track, so wger can fill a category
for you:

* **BMI**, from your body weight and the height in your profile
* **Waist to height ratio**, from a category you measure your waist in and
  the same height
* **One-rep maximum** of an exercise, estimated from the sets you logged for
  it: one value per training day, from the best set of that day
* **The total of several exercises**, the classic being bench press, squat
  and deadlift. Only days on which you trained each of them recently count,
  so the number tracks what you can lift now rather than a personal best

You pick the calculation when you create the category, and wger asks for what
it needs, for example which exercise. Your existing entries are computed
right away, and everything you log afterwards keeps them up to date.

The entries belong to wger, so you cannot edit or delete them: correct the
original instead and the calculated value follows. What a category calculates
is decided when you create it; to stop, delete the category, what the values
were computed from stays untouched. If something wger needs is missing,
usually the height in your profile, the category stays empty.


Charts
------

Your values are drawn as dots, plus a moving average and a trend line, so a
single outlier does not look like a change of direction. When you edit a
category you can set how far the average reaches back and how closely the
trend follows, and turn either line off. Daily totals like sleep or steps
become bars, blood pressure one bar per reading from the diastolic to the
systolic value, and the sleep phases stack into the night.

You can pick the period you look at, and the periods your nutrition plans
were active in are shaded into the chart.

The shape is yours to change as well. Every category comes with the chart
that suits what it measures ("Automatic"), and you can override it when you
edit the category:

* **Line** and **Bars**, when you prefer the other one
* **Heatmap**, a calendar of coloured days that answers "how regularly" more
  than "how much", and where a day you skipped stays visible as an empty cell
* **Change**, one bar per week showing how much the value moved, up or down
* **Distribution**, which sorts your values by size instead of by date and
  shows where most of them lie

Not every shape fits every category, so you only get the ones that make
sense, and the parts of a group are always drawn together.


Sync with Apple Health and Health Connect
-----------------------------------------

If you already record body data with a smart scale, a blood pressure monitor,
a watch or another health app, wger can read it from the platform those apps
write to, so you do not have to type anything in twice. Imported values count
like your own everywhere else: charts, the calendar, the BMI.

Apple Health and Health Connect live on your phone, so the import runs in the
**mobile app**: you turn it on under *Settings → Health* and decide there
which kinds of data to share, declining one of them is fine and the others
are still imported. On Android this needs *Health Connect*, which newer
devices have built in and older ones can install from the store. On iOS it is
Apple Health, which is always there. The values show up on the website as
well, only the import itself needs the phone.

At the moment we import:

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Data
     - How it is stored
   * - Body weight, body fat, lean body mass, height, resting heart rate
     - One entry per reading
   * - Blood pressure
     - One reading, as its systolic and diastolic parts
   * - Heart rate, blood oxygen
     - One entry per day with the average, plus that day's range
   * - Steps, distance, energy burned
     - One entry per day with the total. The energy is only what you burned
       moving, not what your body uses at rest
   * - Sleep
     - One entry per night for the total, plus one per phase when your app
       records them. Nights count towards the day you wake up on

The import runs when you open the app, when you come back to it, and whenever
you tap "sync now" in the settings. The automatic runs are spaced out, so
opening the app twice in a row imports once; "sync now" always runs. That
screen also shows when the last import ran and how much it brought in.


What the sync cannot do
~~~~~~~~~~~~~~~~~~~~~~~

* **Nothing is written back.** wger only reads, so what you enter by hand
  here stays here and never lands in Apple Health or Health Connect.
* **Imported entries cannot be edited in wger**, and correcting or deleting a
  value where it came from does not reach the entry already imported. The
  exception are the entries that summarise a whole day, which are
  recalculated for the past month. Adding your own entries by hand in the
  same categories works as usual.
* **The sync uses its own categories.** If you have been keeping your blood
  pressure in a category you made yourself, the import creates its own
  instead of taking that one over. Body weight is the exception: it goes into
  the one you already use.
* **The history goes back to 2020** at the most, and the first import can
  take a few minutes with the app open.
* **Implausible values are skipped**, which catches a typo like a body
  weight of 7.8 kg as well as the occasional artifact of a wearable.
