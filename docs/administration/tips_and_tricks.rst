.. _tips:

Tips and tricks
---------------

Clearing the cache
~~~~~~~~~~~~~~~~~~

Sometimes there are changes to the internal changes of the cached structures.
It is recommended that you just clear all the existing caches
``python manage.py clear-cache --clear-all`` or just set the timeout to something
like one second (in settings.py::

    CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'wger-cache',
        'TIMEOUT': 1
        }
    }

Miscellaneous settings
~~~~~~~~~~~~~~~~~~~~~~

The following settings can be very useful during development (add to your
settings.py):


**Setting the email backend**
   Use the console backend, all sent emails will be printed to it::

       EMAIL_BACKEND = 'django.core.mail.backends.console.EmailBackend'

Dummy data generator
~~~~~~~~~~~~~~~~~~~~

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


Set the URL for your site in the table ``django_site``. This is only used e.g. in
the password reset emails.



