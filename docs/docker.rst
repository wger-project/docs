Docker images
=============

There are docker files available to quickly get a version of wger up and
running. They are all located under ``extras/docker`` if you want to build
them yourself.

Note that you need to build from the project's source folder, e.g::

    docker build -f extras/docker/development/Dockerfile -t wger/server .
    docker build -f extras/docker/demo/Dockerfile --tag wger/demo .


Production
----------

There is a configured docker compose file with all necessary services

https://github.com/wger-project/docker


Demo
----

Self contained demo image

Get the image::

    docker pull wger/demo

Run a container and start the application::

    docker run -ti --name wger.demo --publish 8000:80 wger/demo


Then just open http://localhost:8000 and log in as: **admin**, password **adminadmin**

Please note that the database will not be persisted when you update the image


Development
-----------

It's also possible to start the development server in docker, without needing to
install any dependencies locally. For this, clone the docker repository https://github.com/wger-project/docker


1. Clone https://github.com/wger-project/wger to a folder of your choice.
2. `cd` into the environment of your choice (dev or dev-postgres)
3. Copy `.env.example` to `.env` and set the path to correspond to the location where
   you have checked out the wger server git repo.

Then::

    docker compose up
    docker compose exec web /bin/bash
    cp extras/docker/development/settings.py .

    # this creates initial db tables, runs yarn install, yarn build:css:sass, etc
    wger bootstrap

    # safe to ignore: Your models in app(s): 'exercises', 'nutrition' have changes that are not yet reflected in a migration, and so won't be applied.
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

You can now login on http://localhost:8000 - there is one user account: admin, with password adminadmin
The server should restart automatically when you change code, etc.


Note: the docker images assume a wger user id of 1000. Since we mount the code
and write from the image into your code repository, you may run into permission errors
if your user id is not 1000. We don't have a good solution for such situation yet.
Check your user id with `echo $UID`.

TODO: document configuring IDE to automatically start the server, debug the python process, view logs, etc.