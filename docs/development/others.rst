.. _others:

Others
===========================

.. _dummy_generator:
Dummy data generator
------------------------

To properly test the different parts of the application for usability or
performance, it is often very useful to have some data to work with. For this
reason, there are dummy data generator scripts for the different entry types::

  dummy-generator-users
  dummy-generator-gyms
  dummy-generator-body-weight
  dummy-generator-nutrition
  dummy-generator-measurement-categories
  dummy-generator-measurements
  dummy-generator-workout-plans
  dummy-generator-workout-diary

Alternatively, you can just call `dummy-generator` which will in turn call the
other scripts with their default values

For help, just do::

  python3 manage.py dummy-generator-<name> --help

.. note::
   All generated users have their username as a password.

.. note::
   While it is possible to generate hundreds of users, gyms are more restricted and
   you will probably get duplicate names if you generate more than a dozen.
