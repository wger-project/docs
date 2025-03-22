.. _development_docker:

Development with docker
========================

You can get a development environment up and running in a few minutes with docker.


Clone https://github.com/wger-project/docker as well as
https://github.com/wger-project/wger and ``cd`` in the docker repo into the
environment of your choice (``dev`` or ``dev-postgres``). Then copy ``.env.example``
to ``.env`` and set the path to correspond to the location where you have checked
out the wger server git repo:


.. code-block:: bash

    git clone https://github.com/wger-project/docker
    git clone https://github.com/wger-project/wger

    cd docker/dev
    cp .env.example .env
    echo "WGER_CODEPATH=/my/wger/path" > .env

Copy in the wger folder the default settings file to the root::

    cp extras/docker/production/settings.py .

Start docker watch in the docker folder:

.. code-block:: bash

    docker compose up --watch

Docker watch will now automatically sync the code changes to and from the container
and rebuild the image when needed. This solves the problem of using a file mount
where the user ID of the host and the container are different and works by having
docker automatically sync the files using the user defined in the docker file.
When you change the requirements.txt file docker automatically rebuilds the image
and restarts the container. When this happens you will probably see some messages
like:

* Syncing service "web" after 1 changes were detected
* Rebuilding service(s) ["web"] after changes were detected...

For more information: https://docs.docker.com/compose/how-tos/file-watch/

Now you can bootstrap the application initially, download the required JS and
CSS libraries and start the development server.


.. code-block:: bash

    docker compose exec web /bin/bash

    # this creates initial db tables, runs yarn install, yarn build:css:sass, etc
    wger bootstrap

    # Alternatively, or if package.json changed, you can also run
    # yarn install
    # yarn build:css:sass

    # safe to ignore: Your models in app(s): 'exercises', 'nutrition' have changes
    # that are not yet reflected in a migration, and so won't be applied.
    python3 manage.py migrate

After this you can run the following commands to load the initial data:

.. code-block:: bash

    # pull exercises from wger.de (or other source you have defined)
    python3 manage.py sync-exercises

    # pull nutrition information
    wger load-online-fixtures

    # if you use sqlite, at this time you can make a backup if you want
    # such that if you mess something up, you don't have to start from scratch
    cp /home/wger/db/database.sqlite /home/wger/db/database.sqlite.orig

    # finally, this is important, start the actual server!
    python3 manage.py runserver 0.0.0.0:8000

You can now login on http://localhost:8000 with the default administrator user
(afterwards you just need to start the server, no need to bootstrap again):

* username: ``admin``
* password: ``adminadmin``


If you use ``dev`` you can use the ``sqlite3`` program to execute queries
against the database file. For ``postgres-sqlite`` you can use
``pgcli -h localhost -p 5432 -u wger`` on your host, with password `wger`


You will probably want to configure your IDE to use the python interpreter
in the container so you can start the server from there. Since this is very
dependent on the one you are using, consult its documentation for how to do
this:

* https://code.visualstudio.com/docs/containers/quickstart-python
* https://www.jetbrains.com/help/pycharm/using-docker-as-a-remote-interpreter.html


Note that while the default configuration for settings.py uses environmental
variables to configure the behaviour of the application, these only take effect
when you restart the container. Since during development you probably want the
changes to take effect immediately, you can just set the values in settings.py,
the development server will pick it up automatically and restart. For example,
in settings.py you will find::

    WGER_SETTINGS["EXERCISE_CACHE_TTL"] = env.int("EXERCISE_CACHE_TTL", 3600)

where the value is read from the env variable EXERCISE_CACHE_TTL. But this can
be changed to just::

    WGER_SETTINGS["EXERCISE_CACHE_TTL"] = 123

to take effect immediately directly.


