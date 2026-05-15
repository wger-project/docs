.. _settings:

Settings
========

wger is configured via environment variables. In a Docker setup, these go in
your env file (typically extending ``prod.env``; see :doc:`/installation/docker`
for the override mechanism). For from-source installations, they go in
``/home/wger/wger.env`` (see :doc:`/installation/from-source`).

The reference ``prod.env`` in the docker repository contains commented
examples for every option and is the canonical source for what's available:
https://github.com/wger-project/docker/blob/master/config/prod.env

This page is the structured reference. For a few specialized topics like S3
storage, SSO via reverse proxy, monitoring, the relevant settings live on
their dedicated pages, linked below.

Required
--------

These should be set explicitly for any non-trivial deployment. Leaving the
defaults means new keys are generated on every restart, which invalidates
sessions and tokens.

``SECRET_KEY``
  Django's secret key, used for cryptographic signing (sessions, CSRF tokens,
  password reset links, etc.). Generate a random 50-character string::

      python -c "import secrets; print(secrets.token_urlsafe(50))"

``SIGNING_KEY``
  JWT signing key for the API. Use a *different* value than ``SECRET_KEY``.
  Same generation method.

``TIME_ZONE`` / ``TZ``
  Server timezone, e.g. ``Europe/Berlin``. See
  https://en.wikipedia.org/wiki/List_of_tz_database_time_zones

``ALLOWED_HOSTS``
  Comma-separated list of hostnames the application accepts requests for,
  e.g. ``example.com,www.example.com``.


``SITE_URL``
  Default ``http://localhost``. Public base URL of your instance, no trailing
  slash. Used to build absolute links in outgoing emails.

CSRF and reverse proxy
----------------------

If you run wger behind a reverse proxy (recommended), Django needs to know
about your domain to accept form submissions. See :doc:`errors` if you run
into CSRF problems.

``CSRF_TRUSTED_ORIGINS``
  Comma-separated list of full URLs (with scheme, and port if non-default),
  e.g. ``https://wger.example.com,https://192.168.1.10:8080``.

``X_FORWARDED_PROTO_HEADER_SET``
  Default ``False``. If your proxy sets the ``X-Forwarded-Proto`` header,
  set this to ``True``. Read the security implications in
  `Django's docs <https://docs.djangoproject.com/en/dev/ref/settings/#secure-proxy-ssl-header>`_.

``NUMBER_OF_PROXIES``
  Default ``1``. Number of proxies in front of the application. Used by
  Django REST Framework's request throttling to determine the real client IP.

URLs and media
--------------

``MEDIA_URL`` / ``STATIC_URL``
  Override the default ``/media/`` and ``/static/`` paths if you serve
  files from a different host (CDN, S3 see :doc:`storage`).

``WGER_PORT``
  Default ``8000``. Port gunicorn binds to inside the container. Rarely
  changed.

Application
-----------

``WGER_INSTANCE``
  Default ``https://wger.de``. Upstream wger instance to sync exercises and
  ingredients from.

``ALLOW_REGISTRATION``
  Default ``True``. Whether users can register accounts on their own.

``ALLOW_GUEST_USERS``
  Default ``True``. Whether the application offers guest user functionality.

``ALLOW_UPLOAD_VIDEOS``
  Default ``True``. Whether users can upload exercise videos.

``MIN_ACCOUNT_AGE_TO_TRUST``
  Default ``21`` (days). Users below this account age cannot contribute to
  exercises.

Database
--------

See :doc:`storage` for switching between Postgres and SQLite.

``DJANGO_DB_ENGINE``
  Default ``django.db.backends.postgresql``. Use ``django.db.backends.sqlite3``
  for SQLite.

``DJANGO_DB_DATABASE``
  Database name (Postgres) or full path to the SQLite file.

``DJANGO_DB_USER``, ``DJANGO_DB_PASSWORD``, ``DJANGO_DB_HOST``, ``DJANGO_DB_PORT``
  Postgres connection details. Ignored for SQLite.

``DJANGO_PERFORM_MIGRATIONS``
  Default ``True``. Apply pending database migrations on container startup.

Cache (Redis)
-------------

``DJANGO_CACHE_BACKEND``
  Default ``django_redis.cache.RedisCache``.

``DJANGO_CACHE_LOCATION``
  Default ``redis://cache:6379/1``. Redis connection URL for the Django cache.

``DJANGO_CACHE_TIMEOUT``
  Default ``1296000`` seconds (15 days).

``DJANGO_CACHE_CLIENT_CLASS``
  Default ``django_redis.client.DefaultClient``.

``DJANGO_CACHE_CLIENT_PASSWORD``
  Optional. Redis password if your Redis is configured with one.

``DJANGO_CACHE_CLIENT_SSL_KEYFILE``, ``DJANGO_CACHE_CLIENT_SSL_CERTFILE``, ``DJANGO_CACHE_CLIENT_SSL_CERT_REQS``, ``DJANGO_CACHE_CLIENT_SSL_CHECK_HOSTNAME``
  Optional TLS settings for Redis connections.

Celery
------

Background task processing. See :doc:`/development/celery` for the worker
setup.

``USE_CELERY``
  Default ``True`` in the Docker setup (``False`` otherwise). Master switch, most
  ``SYNC_*_CELERY`` and ``CACHE_API_*_CELERY`` options require this.

``CELERY_BROKER``, ``CELERY_BACKEND``
  Default ``redis://cache:6379/2``. Redis URL for Celery's broker and
  result backend.

``CELERY_WORKER_CONCURRENCY``
  Default ``4``. Set to ``1`` if using SQLite (which doesn't tolerate
  concurrent writes well).

``CELERY_FLOWER_PASSWORD``
  Default ``adminadmin``. Password for the optional Celery Flower admin UI.

Data sync
---------

See :doc:`sync-data` for the operational context.

**On startup (synchronous, no Celery needed)**

``SYNC_EXERCISES_ON_STARTUP``
  Default ``False``. Sync the exercise database every time the application
  starts. Slows startup; runs synchronously.

``DOWNLOAD_EXERCISE_IMAGES_ON_STARTUP``, ``DOWNLOAD_EXERCISE_VIDEOS_ON_STARTUP``
  Default ``False``. Download exercise images / videos on startup.

``LOAD_ONLINE_FIXTURES_ON_STARTUP``
  Default ``False``. Load a small base set of ingredients on startup.

**Periodic via Celery**

These tasks require ``USE_CELERY=True``. Each picks a random hour and minute
on Celery worker startup and keeps that schedule until the worker restarts,
so two instances won't hammer the upstream wger or Open Food Facts servers
at the same time.

``SYNC_EXERCISES_CELERY``, ``SYNC_EXERCISE_IMAGES_CELERY``, ``SYNC_EXERCISE_VIDEOS_CELERY``
  Default ``True`` in the Docker setup (``False`` otherwise). Run **weekly** on a
  random day of the week.

``CACHE_API_EXERCISES_CELERY``
  Default ``True`` in the Docker setup (``False`` otherwise). Periodically warms up
  the exercise API cache. Runs **weekly** on a random day.

``CACHE_API_EXERCISES_CELERY_FORCE_UPDATE``
  Default ``True``. Always rebuild the cache on each scheduled run, even if
  it looks current.

``SYNC_INGREDIENTS_CELERY``
  Default ``True`` in the Docker setup (``False`` otherwise). Downloads the full
  ingredient dataset as a bulk dump. Runs **monthly** on a random day of
  the month (1–28).

``SYNC_INGREDIENTS_DUMP_URL``
  Default points to the wger.de dump.

``EXPORT_INGREDIENTS_BULK_CELERY``
  Default ``False``. Generates the ingredient bulk dump that can be served
  to other instances. Runs on the **1st and 15th of every month**. Mainly
  used by the wger.de instance itself; you don't normally need to enable
  this.

``SYNC_OFF_DAILY_DELTA_CELERY``
  Default ``False``. Imports Open Food Facts' daily delta product updates.
  Runs **once a day** at a random time. Mainly
  used by the wger.de instance itself; you don't normally need to enable
  this.

``DOWNLOAD_INGREDIENTS_FROM``
  Default ``WGER`` (or ``None`` to disable). Where to fetch ingredient data
  from when scanning a barcode for an unknown product. Requires Celery.

**Always-on**

When ``USE_CELERY=True``, expired JWT refresh tokens are flushed from the
database **once a day** at a random time. There is no opt-out, without it
the token table grows monotonically.

JWT authentication
------------------

``ACCESS_TOKEN_LIFETIME``
  Default ``10`` (minutes). Short-lived access token.

``REFRESH_TOKEN_LIFETIME``
  Default ``2880`` (hours, ≈ 4 months). Long-lived refresh token.

For SSO via reverse proxy (``AUTH_PROXY_*``), see :doc:`auth_proxy`.

Brute-force protection
----------------------

Powered by `django-axes <https://django-axes.readthedocs.io/>`_.

``AXES_ENABLED``
  Default ``True``.

``AXES_FAILURE_LIMIT``
  Default ``10``. Failed login attempts before lockout.

``AXES_COOLOFF_TIME``
  Default ``30`` (minutes). Lockout duration.

``AXES_HANDLER``
  Default ``axes.handlers.cache.AxesCacheHandler``.

``AXES_LOCKOUT_PARAMETERS``
  Default ``ip_address``. What to lock on (IP, username, etc.).

``AXES_IPWARE_PROXY_COUNT``, ``AXES_IPWARE_META_PRECEDENCE_ORDER``
  Advanced settings for IP detection behind proxies. See django-axes docs.

reCAPTCHA
---------

``USE_RECAPTCHA``
  Default ``False``. Show a captcha challenge during user registration.

``RECAPTCHA_PUBLIC_KEY``, ``RECAPTCHA_PRIVATE_KEY``
  Your keys from https://www.google.com/recaptcha/

``RECAPTCHA_REQUIRED_SCORE``
  Default ``0.75``.

.. _email:

Email
-----

By default wger uses Django's console email backend (writes outgoing emails
to stdout). To use a real SMTP server:

``ENABLE_EMAIL``
  Default ``False``. Set to ``True`` to enable real email delivery.

``EMAIL_HOST``, ``EMAIL_PORT``
  SMTP server hostname and port (commonly ``587``).

``EMAIL_HOST_USER``, ``EMAIL_HOST_PASSWORD``
  SMTP credentials.

``EMAIL_USE_TLS``, ``EMAIL_USE_SSL``
  Default ``True`` and ``False``. Pick whichever your provider uses
  (typically TLS for port 587, SSL for port 465).

``FROM_EMAIL``
  Default ``wger Workout Manager <wger@example.com>``. The "From:" address
  on outgoing emails.

``DJANGO_ADMINS``
  Optional. ``Name,email@example.com`` to receive notifications about
  internal server errors. Requires a working email configuration.

To verify your setup, run::

    python manage.py sendtestemail user@example.com

(In Docker: ``docker compose exec web python3 manage.py sendtestemail …``)

Logging
-------

``LOG_LEVEL_PYTHON``
  Default ``INFO``. Possible values: ``DEBUG``, ``INFO``, ``WARNING``,
  ``ERROR``, ``CRITICAL``.

``DJANGO_DEBUG``
  Default ``False``. **Never enable in production**, Django leaks internal
  details (stack traces, settings) on error pages.

Static files
------------

For S3-backed static and media files, see :doc:`storage`.

``DJANGO_COLLECTSTATIC_ON_STARTUP``
  Default ``True``. Run ``collectstatic`` on every container start. Set to
  ``False`` for faster startup if you call collectstatic manually.

``DJANGO_CLEAR_STATIC_FIRST``
  Default ``False``. Delete the static directory before re-collecting. Useful
  if stale files are causing problems after upgrades.

``COMPRESS_ENABLED``
  Optional. Enables django-compressor to bundle CSS/JS into single files.

Gunicorn
--------

``WGER_USE_GUNICORN``
  Default ``True`` in production, ``False`` in dev (uses Django's
  development server).

``GUNICORN_CMD_ARGS``
  Default ``--workers 3 --threads 2 --worker-class gthread --timeout 240``.
  A common starting point for worker count is ``(2 × $num_cores) + 1``.
  See https://docs.gunicorn.org/en/stable/settings.html

Monitoring
----------

``EXPOSE_PROMETHEUS_METRICS``
  Default ``False``. Expose Prometheus metrics endpoints. See :doc:`monitoring`.

Python-only settings
--------------------

Docker users can skip this section; everything common is exposed as an env
var above. If you maintain a custom settings module (see
:doc:`/development/backend`), there are two cache settings that can only be
set in Python via the ``WGER_SETTINGS`` dictionary:

.. code-block:: python

    WGER_SETTINGS['INGREDIENT_CACHE_TTL'] = 86400
    WGER_SETTINGS['ROUTINE_CACHE_TTL'] = 86400

``INGREDIENT_CACHE_TTL``
  Default ``604800`` (one week). How long ingredient list responses are cached.

``ROUTINE_CACHE_TTL``
  Default ``2419200`` (four weeks). How long routine list responses are cached.

.. _site-settings:

Site settings (Django sites framework)
--------------------------------------

Some wger features use Django's ``contrib.sites`` framework for the site
domain and name. The Docker entrypoint sets these from ``SITE_URL``
automatically via the ``set-site-url`` management command. For from-source
installations or to override manually, run::

    python manage.py set-site-url https://wger.example.com

Or, equivalently, through the Python shell::

    python manage.py shell
    >>> from django.contrib.sites.models import Site
    >>> site = Site.objects.get(pk=1)
    >>> site.domain = 'wger.example.com'
    >>> site.name = 'example.com wger Workout Manager'
    >>> site.save()

This assumes the default site ID of 1. If you use a different one, set it
in your settings module::

    SITE_ID = 2

Template customization
----------------------

If you run a public instance, you might want or need to customize a couple of
templates to match info and legal requirements:

* ``wger/software/templates/tos.html`` for your Terms of Service
* ``wger/core/templates/misc/about.html`` for your contact address or other
  legal information

With Docker, the simplest option is to mount your replacements over the
files inside the container via your ``docker-compose.override.yml``.

.. code-block:: yaml

    services:
      web:
        volumes:
          - ./my-tos.html:/home/wger/src/wger/software/templates/tos.html:ro
          - ./my-about.html:/home/wger/src/wger/core/templates/misc/about.html:ro

For other setups, these are standard Django templates. See `Django's docs on
overriding templates <https://docs.djangoproject.com/en/dev/howto/overriding-templates/>`_
for the general approach.
