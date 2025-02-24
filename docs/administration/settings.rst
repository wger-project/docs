.. _settings:

Settings
========

You can configure some of the application behaviour with the ``WGER_SETTINGS``
dictionary in your settings file. Currently, the following options are supported:

**ALLOW_GUEST_USERS**: Default ``True``.
  Controls whether users can use the site as a guest user or if an administrator
  has to create the user accounts, as with the option above.

**ALLOW_REGISTRATION**: Default ``True``.
  Controls whether users can register on their own or if a gym administrator has
  to create the user accounts.

**DOWNLOAD_INGREDIENTS_FROM**: Default ``WGER``
  Where to download ingredient images from. Set to 'None' to deactivate.

**EMAIL_FROM**: Default ``wger Workout Manager <wger@example.com>``
  The sender address used for sent emails by the system such as weight reminders

**EXERCISE_CACHE_TTL**: Default ``3600``
  Sets how long the overview responses for exercise, exerciseinfo and exercisebaseinfo
  are cached for. The value is in seconds, so 3600 is one hour.

**MIN_ACCOUNT_AGE_TO_TRUST**: Default ``21``
  Users won't be able to contribute to exercises if their account age is
  lower than this amount in days.

**SYNC_EXERCISES_CELERY**: Default ``False``
  Whether to periodically synchronize the exercise database from the default wger
  instance. Needs celery to be configured.

**SYNC_EXERCISE_IMAGES_CELERY**: Default ``False``
  Whether to periodically synchronize the exercise images from the default wger
  instance. Needs celery to be configured.

**SYNC_EXERCISE_VIDEOS_CELERY**: Default ``False``
  Whether to periodically synchronize the exercise videos from the default wger
  instance. Needs celery to be configured.

**USE_CELERY**: Default ``False``.
  Whether celery is configured and can be used

**USE_RECAPTCHA**: Default ``False``.
  Controls whether a captcha challenge will be presented when new users register.

**WGER_INSTANCE**: Default ``https://wger.de``.
  The wger instance from which commands like exercise synchronization will use
  to fetch data from.


.. note::
  If you want to override a default setting, don't overwrite all the dictionary
  but only the keys you need, e.g. ``WGER_SETTINGS['foo'] = 'bar'``. This avoids
  problems when new keys are added in the global settings.
