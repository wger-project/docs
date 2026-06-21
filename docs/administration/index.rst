.. _administration:

Administration
==============

This section describes configurations, commands and operations relevant for
administrators running a wger instance.

.. note::

   Most examples on the following pages assume a Docker compose setup, since
   that is the recommended deployment. If you installed wger differently, just
   adapt the commands to your environment (e.g., ``docker compose exec web ./manage.py …``
   might become ``./manage.py …``).

.. toctree::
   :maxdepth: 1
   :hidden:

   lifecycle
   updating
   backup
   sync-data
   monitoring
   settings
   commands
   errors
   storage
   powersync
   auth_proxy
   social_auth
   mfa
   anubis
   postgres
   gym

:doc:`lifecycle`
    Stop, start, inspect, and auto-start the application via systemd.

:doc:`updating`
    Update wger to a new release.

:doc:`backup`
    Backup and restore your wger instance.

:doc:`sync-data`
    Sync exercises and ingredients from upstream.

:doc:`monitoring`
    Monitor the application with Grafana and Prometheus.

:doc:`settings`
    Available configuration options for the application.

:doc:`commands`
    Reference for the available administration and management commands.

:doc:`errors`
    Common errors and how to fix them.

:doc:`storage`
    Database backend (Postgres, SQLite) and S3-compatible object storage
    for media and static files.

:doc:`powersync`
    Setup and maintenance for the PowerSync sync service

:doc:`auth_proxy`
    Delegate authentication to a reverse proxy or external SSO provider.

:doc:`social_auth`
    Social authentication (OAuth2) via Google, GitHub, GitLab, and other providers.

:doc:`mfa`
    Two-factor authentication (TOTP, recovery codes) and passkeys (WebAuthn).

:doc:`anubis`
    Protect public-facing instances from AI scrapers.

:doc:`postgres`
    Upgrade Postgres to a newer major version.

:doc:`gym`
    Manage gyms, members, trainers and contracts.
