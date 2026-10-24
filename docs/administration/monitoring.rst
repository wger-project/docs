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

Alerts
------

Grafana can send you an email when something goes wrong. The following alerts
are included:

* a background task (e.g. the daily ingredient update) failed
* no background tasks were scheduled or finished for 26 hours, which usually
  means the ``celery_beat`` or ``celery_worker`` container is not running
* a service that Prometheus collects metrics from could not be reached for
  5 minutes

To receive the emails, set the mail server in ``grafana/.env`` (see the
``GRAFANA_SMTP_*`` entries in ``grafana/.env.example``) and change the
recipient address in ``grafana/provisioning/alerting/contactpoints.yml``.
Then restart the Grafana compose file. The alerts are listed in Grafana under
*Alerting > Alert rules*, in the *Alerts* folder.
