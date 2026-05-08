.. _backup:

Backup
======

The most important thing to back up is the **Postgres database** — that's where
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

Media
-----

If you haven't uploaded your own exercise images, exercise videos or gallery
images, you don't need to back up the media volume — the contents can be
re-downloaded from the upstream wger instance. Truncate the relevant tables
and run the sync commands again:

.. code-block:: bash

    docker compose exec db psql -U wger "TRUNCATE TABLE exercises_exerciseimage, exercises_exercisevideo;"
    docker compose exec db psql -U wger "TRUNCATE TABLE nutrition_image;"

    docker compose exec web python3 manage.py download-exercise-images
    docker compose exec web python3 manage.py download-exercise-videos

If you do have uploaded media that needs to be preserved, consult these
options for backing up Docker volumes:

* https://www.docker.com/blog/back-up-and-share-docker-volumes-with-this-extension/
* https://github.com/BretFisher/docker-vackup

Static files
------------

The contents of the static volume are 100% generated and recreated on startup —
no need to back up anything.
