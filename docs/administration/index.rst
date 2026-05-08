.. _administration:

Administration
==============

This section describes configurations, commands and operations relevant for
administrators running a wger instance.

.. note::

   Most examples on the following pages assume a Docker compose setup, since
   that is the recommended and most extensively tested deployment. If you
   installed wger from source or via a community script, the concepts apply
   unchanged — adapt the commands to your environment (e.g.,
   ``docker compose exec web ./manage.py …`` becomes ``./manage.py …``
   directly in your activated virtualenv).

.. toctree::
   :maxdepth: 1
   :hidden:

   errors
   commands
   settings
   auth_proxy
   lifecycle
   backup
   postgres
   monitoring
   gym

:doc:`errors`
    Common errors and how to fix them in a production environment.

:doc:`commands`
    Reference for the available administration and management commands.

:doc:`settings`
    Available configuration options for the application.

:doc:`auth_proxy`
    Delegate authentication to a reverse proxy or external SSO provider.

:doc:`lifecycle`
    Stop, start, inspect, and auto-start the application via systemd.

:doc:`backup`
    Backup and restore your wger instance.

:doc:`postgres`
    Upgrade Postgres to a newer major version.

:doc:`monitoring`
    Monitor the application with Grafana and Prometheus.

:doc:`gym`
    Manage gyms, members, trainers and contracts.
