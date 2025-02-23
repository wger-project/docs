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


Selectively running tests
~~~~~~~~~~~~~~~~~~~~~~~~~

If you do a ``python manage.py test`` you will run the complete testsuite, and
this can take a while. You can control which tests will be executed like this.

Test only the tests in the 'core' app::

  python manage.py test wger.core

Test only the tests in the 'test_user.py` file in the core app::

  python manage.py test wger.core.tests.test_user

Test only the tests in 'StatusUserTestCase' in the file 'test_user.py` file in
the core app::

  python manage.py test wger.core.tests.test_user.StatusUserTestCase


Using runserver_plus
~~~~~~~~~~~~~~~~~~~~

During development, you can use ``runserver_plus`` instead of the default Django
server as you can use an interactive debugger directly from the browser if an
exception occurs. It also accepts the same command-line options. For this just
install the following packages::

    pip install django_extensions werkzeug
    python manage.py runserver_plus [options]


Contributing
============

* **Send pull requests**: for new code you want to share, please send pull
  requests in GitHub. Sending patches by email or attaching them to an issue
  means a lot more work. It's recommended that you work on a feature branch
  when working on something, especially when it's something bigger. While many
  people insist on rebasing before sending a pull request, it's not necessary.

* **Run the tests**: wger is proud to have a test coverage of over 90%. When you
  implement something new, don't forget to run the testsuite and write appropriate
  tests for the new code.

