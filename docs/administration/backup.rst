.. _backup:

Backup
======

The most important thing to back up is the **Postgres database**, that's where
all user data lives. The media volume can usually be re-fetched from upstream,
and static files are regenerated on every container start.

You should perform a test restore at least once, so you know the procedure works
when you actually need it.

Database
--------

Make a dump of the database:

.. code-block:: bash

    # Stop the other services so the database isn't changed mid-export
    docker compose stop web nginx cache celery_worker celery_beat
    docker compose exec db pg_dumpall --clean --username wger > backup.sql
    docker compose start

To restore from a dump:

.. code-block:: bash

    docker compose stop
    docker volume remove docker_postgres-data
    docker compose up db
    cat backup.sql | docker compose exec -T db psql --username wger --dbname wger
    docker compose up

Automated backups
~~~~~~~~~~~~~~~~~

The docker repository contains a script and a systemd timer for daily dumps in
the ``backup`` folder. The script dumps the database in PostgreSQL's custom
format and checks that the dump can be read. By default, we keep 7 daily,
4 weekly, and 6 monthly dumps. The database keeps running while the backup is made.

.. code-block:: bash

    # Run once by hand
    ./backup/backup-db.sh

    # Install the daily timer (adjust the path in the service file first)
    cp backup/wger-backup.service backup/wger-backup.timer /etc/systemd/system/
    systemctl daemon-reload
    systemctl enable --now wger-backup.timer

The settings (backup folder, how long dumps are kept) are listed at the top
of ``backup-db.sh``. It is recommended to keep at least one copy outside the server,
a backup on the same disk does not help if the server is lost.

To restore such a dump into a fresh database:

.. code-block:: bash

    docker compose down
    docker volume remove docker_postgres-data
    docker compose up -d --wait db
    docker compose exec -T db pg_restore --username wger --dbname wger \
        --no-owner < /root/dumps/daily/wger-<date>.dump
    docker compose up -d
    docker compose exec web ./manage.py setup-powersync-storage

The dump contains neither the ``powersync`` schema nor database roles, so the
last command creates them again.

PowerSync
~~~~~~~~~

The PowerSync bucket-storage tables (``powersync.*`` schema) are effectively
a **cache** rebuilt from the ``public.*`` wger tables and ``sync_rules.yaml``.
Nothing in there is a primary source of truth, every row can be regenerated
via a fresh snapshot against the main wger database.

``pg_dumpall`` above captures the schema automatically, so there is
nothing extra to do. If backup size matters (``bucket_data`` scales
with users × sync-rule complexity), you can safely exclude it::

   docker compose exec db pg_dump --username wger \
       --exclude-schema=powersync --clean --create wger > backup.sql

On restore, PowerSync re-bootstraps the schema on next startup and
takes a fresh snapshot. Clients silently re-sync their local SQLite
from scratch.

.. important::

   **After restoring the database**, drop the PowerSync replication slot and
   restart the service so it re-reads from a valid WAL position:

   .. code-block:: bash

       docker compose exec db psql -U wger -c \
           "SELECT pg_drop_replication_slot(slot_name) \
            FROM pg_replication_slots \
            WHERE slot_name LIKE 'powersync_%';"
       docker compose restart powersync

   Skipping this leaves the slot pointing at a WAL position that no longer
   exists, and PowerSync fails with cryptic ``operator does not exist`` errors
   until it is re-created.

Media
-----

If you haven't uploaded your own exercise images, exercise videos or gallery
images, you don't need to back up the media volume, the contents can be
re-downloaded from the upstream wger instance. Truncate the relevant tables
and run the sync commands again:

.. code-block:: bash

    docker compose exec db psql -U wger -c "TRUNCATE TABLE exercises_exerciseimage, exercises_exercisevideo;"
    docker compose exec db psql -U wger -c "TRUNCATE TABLE nutrition_image;"

    docker compose exec web python3 manage.py download-exercise-images
    docker compose exec web python3 manage.py download-exercise-videos

If you do have uploaded media that needs to be preserved, such as gallery entries,
consult these options for backing up Docker volumes:

* https://www.docker.com/blog/back-up-and-share-docker-volumes-with-this-extension/
* https://github.com/BretFisher/docker-vackup

Static files
------------

The contents of the static volume are 100% generated and recreated on startup,
no need to back up anything.
