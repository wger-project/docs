.. _development_docker:
Development with docker
========================

Clone https://github.com/wger-project/wger to a folder of your choice and
``cd`` into the environment of your choice (dev or dev-postgres)

Copy ``.env.example`` to ``.env`` and set the path to correspond to the location where you
have checked out the wger server git repo.

.. code-block:: bash

    docker compose up
    docker compose exec web /bin/bash
    cp extras/docker/production/settings.py .

    # this creates initial db tables, runs yarn install, yarn build:css:sass, etc
    wger bootstrap

    # safe to ignore: Your models in app(s): 'exercises', 'nutrition' have changes
    # that are not yet reflected in a migration, and so won't be applied.
    python3 manage.py migrate

    # pull exercises from wger.de (or other source you have defined)
    python3 manage.py sync-exercises

    # pull nutrition information
    wger load-online-fixtures

    # if you use sqlite, at this time you can make a backup if you want
    # such that if you mess something up, you don't have to start from scratch
    cp /home/wger/db/database.sqlite /home/wger/db/database.sqlite.orig

    # finally, this is important, start the actual server!
    python3 manage.py runserver 0.0.0.0:8000

You can now login on http://localhost:8000 - there is one user account: admin,
with password adminadmin. The server should restart automatically when you
change code, etc.

If you use ``dev`` you can use the ``sqlite3`` program to execute queries
against the database file. For ``postgres-sqlite`` you can use
``pgcli -h localhost -p 5432 -u wger`` on your host, with password `wger`


Building the images
-------------------

You can easily build your own docker images, just run these commands from
the server's source folder:

.. code-block:: bash

    docker build -f extras/docker/development/Dockerfile -t wger/server .
    docker build -f extras/docker/demo/Dockerfile --tag wger/demo .