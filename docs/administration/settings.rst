.. _settings:

Settings
========

WGER_SETTINGS dictionary
------------------------

You can configure some of the application behaviour with the ``WGER_SETTINGS``
dictionary in your settings file. Currently, the following options are supported:

``ALLOW_GUEST_USERS``
  Default ``True``

  Controls whether users can use the site as a guest user or if an administrator
  has to create the user accounts, as with the option above.

``ALLOW_REGISTRATION``
  Default ``True``

  Controls whether users can register on their own or if a gym administrator has
  to create the user accounts.

``DOWNLOAD_INGREDIENTS_FROM``
  Default ``WGER``

  Where to download ingredient images from. Set to 'None' to deactivate.

``EMAIL_FROM``
  Default ``wger Workout Manager <wger@example.com>``

  The sender address used for sent emails by the system such as weight reminders

``EXERCISE_CACHE_TTL``
  Default ``3600`` (one hour)

  Sets how long the overview responses for exercise, exerciseinfo and exercisebaseinfo
  are cached for.

``INGREDIENT_CACHE_TTL``
  Default ``604800`` (one week)

  Sets how long the overview responses for ingredients are cached for.

``ROUTINE_CACHE_TTL``
  Default ``4 * 604800`` (four weeks)

  Sets how long the overview responses for routines are cached for.

``MIN_ACCOUNT_AGE_TO_TRUST``
  Default ``21``

  Users won't be able to contribute to exercises if their account age is
  lower than this amount in days.

``SYNC_EXERCISES_CELERY``
  Default ``False``

  Whether to periodically synchronize the exercise database from the default wger
  instance. Needs celery to be configured.

``SYNC_EXERCISE_IMAGES_CELERY``
  Default ``False``

  Whether to periodically synchronize the exercise images from the default wger
  instance. Needs celery to be configured.

``SYNC_EXERCISE_VIDEOS_CELERY``
  Default ``False``

  Whether to periodically synchronize the exercise videos from the default wger
  instance. Needs celery to be configured.

``USE_CELERY``
  Default ``False``.

  Whether celery is configured and can be used

``USE_RECAPTCHA``
  Default ``False``.

  Controls whether a captcha challenge will be presented when new users register.

``WGER_INSTANCE``
  Default ``https://wger.de``.

  The wger instance from which commands like exercise synchronization will use
  to fetch data from.


.. note::
  If you want to override a default setting, don't overwrite all the dictionary
  but only the keys you need, e.g. ``WGER_SETTINGS['foo'] = 'bar'``. This avoids
  problems when new keys are added in the global settings.

.. _email:

Email
-----

The application is configured to use Django's console email backend by default,
which causes messages intended to be sent via email to be written to ``stdout``.

In order to use a real email server, another backend listed in
`Django's documentation`_ can be configured instead. Parameters for the backend
are set as variables in ``settings.py``. For example, the following allows an
SMTP server at ``smtp.example.com`` to be used:

.. code-block:: bash

   export ENABLE_EMAIL = True
   export EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
   export EMAIL_HOST = 'smtp.example.com'
   export EMAIL_PORT = 587
   export EMAIL_HOST_USER = 'wger@example.com'
   export EMAIL_HOST_PASSWORD = 'example_password'
   export EMAIL_USE_TLS = True
   export EMAIL_USE_SSL = False
   export FROM_EMAIL = 'wger Workout Manager <wger@example.com>'

Django provides a ``sendtestemail`` command via ``manage.py`` to test email
settings::

  python manage.py sendtestemail user@example.com

.. _`Django's documentation`: https://docs.djangoproject.com/en/dev/topics/email/#email-backends

.. _site-settings:

Site settings
-------------

Some wger features make use of Django's site name and domain settings in the
``contrib.sites`` framework. These should be set through the Python shell::

   python manage.py shell
   >>> from django.contrib.sites.models import Site
   >>> site = Site.objects.get(pk=1)
   >>> site.domain = 'wger.example.com'
   >>> site.name = 'example.com wger Workout Manager'
   >>> site.save()

where ``wger.example.com`` is the domain of the wger instance. This assumes
that wger is using the default site ID of 1. If a different site ID is being
used, it must be specified in ``settings.py``::

  SITE_ID = 2
