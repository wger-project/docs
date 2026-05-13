.. _monitoring:

Monitoring
==========

The docker-compose repository ships with a pre-configured Grafana and
Prometheus setup that can be used to monitor the wger application, as well as
the logs via Loki and Alloy:

1. Set ``EXPOSE_PROMETHEUS_METRICS=True`` in your env file and restart the
   application.
2. Change into the ``grafana`` folder of the docker-compose repository and
   start its compose file.
3. Open http://localhost:3000 and log in with username ``admin`` and password
   ``adminadmin``.

To change the default password, edit ``grafana/web.yml``.
