.. _docker_prod:

Docker compose
==============

https://github.com/wger-project/docker

The prod docker compose file starts up a production environment with gunicorn
as the webserver, postgres as a database and redis for caching with nginx
used as a reverse proxy.

The database, static files and uploaded images are mounted as volumes so
the data is persisted. The only thing you need to do is update the docker
images. Consult the docker volume command for details on how to access or
backup this data.

It is recommended to regularly pull the latest version of the compose file,
since sometimes new configurations or environmental variables are added.

If after installing not everything works, consult the :ref:`errors_and_pitfalls`
section for common errors and how to fix them.

Other setups
------------

  **Kubernetes**
    There is a helm charts repository if you want to deploy the application with
    kubernetes: https://github.com/wger-project/helm-charts

  **TrueNAS SCALE**
    Consult this guide if you want to deploy the application to :ref:`truenas`

Quickstart
----------
To start all services::

    docker compose up -d

Then open http://localhost (or your server's IP) and log in as: **admin**,
password **adminadmin**.

**Updating exercises and ingredients**

The docker image comes with a default set of exercises, but due to the size of
the dataset, no ingredients. You can manually sync the datasets from the wger.de
instance (or any other) with the following commands:

.. code-block:: bash

    docker compose exec web python3 manage.py sync-exercises
    docker compose exec web python3 manage.py download-exercise-images
    docker compose exec web python3 manage.py download-exercise-videos

The full ingredient dataset is quite larger, taking around 1GB of space in the
db and needs a far longer time to download and process:

.. code-block:: bash

    # (quickly) loads a base set of ingredients
    docker compose exec web wger load-online-fixtures

    # Downloads the full ingredient dataset, alternatively in the background
    docker compose exec web python3 manage.py sync-ingredients
    docker compose exec web python3 manage.py sync-ingredients-async

The application is configured to perform these steps in the background, but you
can turn them off by changing the ``SYNC_*`` options in ``prod.env``.

Also note that these sync commands will not overwrite any exercises you might
have added yourself to your instance.

**Update the application**

Just remove the containers and pull the newest version:

.. code-block:: bash

    docker compose down
    docker compose pull
    docker compose up -d

**Lifecycle Management**

To stop all services issue a stop command, this will preserve all containers
and volumes::

    docker compose stop

To start everything up again::

    docker compose start

To remove all containers (except for the volumes)::

    docker compose down

To view the logs::

    docker compose logs -f

You might need to issue other commands or do other manual work in the container,
e.g.:

.. code-block:: bash

     docker compose exec web python3 manage.py migrate
     docker compose exec --user root web /bin/bash
     docker compose exec db psql wger -U wger
     docker compose exec cache redis-cli FLUSHALL

Configuration
-------------

Instead of editing the compose file or the env file directly, it is recommended
to extend it. That way you can more easily pull changes from this repository.

For example, you might not want to run the application on port 80 because some
other service in your network is already using it. For this, simply create a new
file called ``docker-compose.override.yml`` with the following content

.. code-block:: yaml

    services:
      nginx:
        ports:
          - "8080:80"

Now the port setting will be overwritten from the configured nginx service when
you do a ``docker compose up``. However, note that compose will concatenate both sets
of values so in this case the application will be binded to 8080 (from the override)
*and* 80 (from the regular compose file). 

In Docker Compose 2.24.4 and later, you can fully override values using the !override 
yaml directive. i.e.:

.. code-block:: yaml

    services:
      nginx:
        ports: !override
          - "8080:80"

The same applies to the env variables, just create a new file called e.g. ``my.env``
and add it after the provided ``prod.env`` for the web service (again, this is
``docker-compose.override.yml``). There you add the settings that you changed, and only
those, which makes it easier to troubleshoot, etc.

.. code-block:: yaml

    web:
      env_file:
        - ./config/prod.env
        - ./config/my.env

To add a web interface for the celery queue, add a new service to the override file

.. code-block:: yaml

    celery_flower:
      image: wger/server:latest
      container_name: wger_celery_flower
      command: /start-flower
      env_file:
        - ./config/prod.env
      ports:
        - "5555:5555"
      healthcheck:
        test: wget --no-verbose --tries=1 http://localhost:5555/healthcheck
        interval: 10s
        timeout: 5s
        retries: 5
      depends_on:
        celery_worker:
          condition: service_healthy

For more information and possibilities consult https://docs.docker.com/compose/extends/ and https://docs.docker.com/reference/compose-file/merge/

Deployment
----------

The easiest way to deploy this application to prod is to use a reverse proxy like
nginx or traefik. You can change the port this application exposes and reverse proxy
your domain to it. For this just edit the "nginx" service in docker-compose.yml and
set the port to some value, e.g. ``"8080:80"`` then configure your proxy to forward
requests to it, e.g. for nginx (no other ports need to be changed, they are used
only within the application's docker network).

There is also an example with Caddy, a webserver that can automatically generate
SSL certificates for you and is very easy to use.

Also notice that the application currently needs to run on its own (sub)domain
and not in a subdirectory, so ``<domain>/wger`` will probably only mostly work.

Monitoring with grafana
-----------------------

There's a pre-configured grafana and prometheus setup that can be used to monitor
the wger application as well as the logs with Loki and Alloy. To start, set the
``EXPOSE_PROMETHEUS_METRICS`` to true in the env file and restart the application,
then go into the ``grafana`` folder and start the compose file.

To access the dashboards, go to http://localhost:3000 and log in with ``admin``, password
``adminadmin``. To change the pre defined password, edit ``grafana/web.yml``.

Others
------


Automatically start service
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If everything works correctly, you will want to start the compose file as a
service so that it auto restarts when you reboot the server. If you use systemd,
this can be done with a simple file. Create the file ``/etc/systemd/system/wger.service``
and enter the following content (check where the absolute path of the docker
command is with ``which docker``)

.. code-block:: ini

    [Unit]
    Description=wger docker compose service
    PartOf=docker.service
    After=docker.service

    [Service]
    Type=oneshot
    RemainAfterExit=true
    WorkingDirectory=/path/to/the/docker/compose/
    ExecStart=/usr/bin/docker compose up -d --remove-orphans
    ExecStop=/usr/bin/docker compose down

    [Install]
    WantedBy=multi-user.target

Read the file with ``systemctl daemon-reload`` and start it with
``systemctl start wger``. If there are no errors and ``systemctl status wger``
shows that the service is active (this might take some time), everything went
well. With ``systemctl enable wger`` the service will be automatically restarted
after a reboot.

Backup
~~~~~~

**Database volume:** The most important thing to backup. For this just make
a dump and restore it when needed (you should try to run this once to make
sure it works).:

.. code-block:: bash

    # Stop all other containers so the db is not changed while you export it
    docker compose stop web nginx cache celery_worker celery_beat
    docker compose exec db pg_dumpall --clean --username wger > backup.sql
    docker compose start

    # When you need to restore it
    docker compose stop
    docker volume remove docker_postgres-data
    docker compose up db
    cat backup.sql | docker compose exec -T db psql --username wger --dbname wger
    docker compose up

**Media volume:** If you haven't uploaded  to exercises or the gallery, you
don't need to backup this, the contents can just be downloaded again.
Just delete any data in the appropriate tables and run the sync commands again:

.. code-block:: bash

    docker compose exec db psql -U wger "TRUNCATE TABLE exercises_exerciseimage, exercises_exercisevideo;";
    docker compose exec db psql -U wger "TRUNCATE TABLE nutrition_image;";

    docker compose exec web python3 manage.py download-exercise-images
    ...


If you have, please consult these possibilities:

* https://www.docker.com/blog/back-up-and-share-docker-volumes-with-this-extension/
* https://github.com/BretFisher/docker-vackup


**Static volume:** The contents of this volume are 100% generated and recreated
on startup, no need to backup anything

Postgres Upgrade
~~~~~~~~~~~~~~~~

It is sadly not possible to automatically upgrade between postgres versions,
you need to perform the upgrade manually. Since the amount of data the app
generates is small a simple dump and restore is the simplest way to do this.

If you pulled new changes from this repo and got the error message "The data
directory was initialized by PostgreSQL version 12, which is not compatible
with this version 15." this is for you.

See also https://github.com/docker-library/postgres/issues/37

.. code-block:: bash

    # Checkout the last version of the composer file that uses postgres 12
    git checkout pg-12

    # Stop all other containers
    docker compose stop web nginx cache celery_worker celery_beat

    # Make a dump of the database and remove the container and volume
    docker compose exec db pg_dumpall --clean --username wger > backup.sql
    docker compose stop db
    docker compose down
    docker volume remove docker_postgres-data

    # Checkout current version, import the dump and start everything
    git checkout master
    docker compose up db
    cat backup.sql | docker compose exec -T db psql --username wger --dbname wger
    docker compose exec -T db psql --username wger --dbname wger -c "ALTER USER wger WITH PASSWORD 'wger'"
    docker compose up
    rm backup.sql

Building the image
~~~~~~~~~~~~~~~~~~

If you want to build your own image, you can do so by running the following
commands from the server's source folder:

.. code-block:: bash

    docker build -f extras/docker/development/Dockerfile -t wger/server .


There is also a "base" image located in ``extras/docker/base`` which the
server one uses as a base.

Using sqlite
~~~~~~~~~~~~

You can also easily use sqlite as a database backend instead of postgres. If you
want to use an existing database, copy it to next to the docker compose file,
otherwise create an empty file, make sure the permissions are correct and then
change these settings::

  touch ./database.sqlite
  chmod 664 ./database.sqlite

In the env file::

  DJANGO_DB_ENGINE=django.db.backends.sqlite3
  DJANGO_DB_DATABASE=/home/wger/db/database.sqlite

In the ``docker-compose.yml`` file, change the volume mapping for the web and celery
services, remove the dependency on the db service and remove the entire db service
definition:

.. code-block:: yaml

    web:
      image: docker.io/wger/server:latest
      depends_on:
        # delete this
        db:
          condition: service_healthy
        [...]
      volumes:
        - ./database.sqlite:/home/wger/db/database.sqlite
        [...]

    celery_worker:
      image: docker.io/wger/server:latest
      volumes:
        - ./database.sqlite:/home/wger/db/database.sqlite
        [...]

    # remove the db service
    db:
      [...]

S3 / Object Storage
~~~~~~~~~~~~~~~~~~~

Wger, or rather django, supports serving static (JS, CSS, etc.) as well as media
(gallery images, exercise and ingredient images, etc.) files from an S3-compatible
object store.

Note that if you enable this, you don't need nginx to serve these files, you can
just reverse proxy the application. Also note that you will probably need to edit
the CORS settings in the storage provider to allow the application to access the
files from the S3 domain.

``USE_S3_MEDIA_FILES``
  bool, default: ``False``

  Enable S3-backed media storage.

``USE_S3_STATIC_FILES``
  bool, default: ``False``

  Enable S3-backed static files storage. Note that if you enable this, it's
  recommended to set ``DJANGO_COLLECTSTATIC_ON_STARTUP``to false since the collectstatic
  command will needlessly increase the startup time.

``AWS_ACCESS_KEY_ID``
  access key

``AWS_SECRET_ACCESS_KEY``
  secret key

``AWS_STORAGE_BUCKET_NAME``
  bucket name

``AWS_S3_REGION_NAME``
  region used to build endpoints such as ``eu-central-1`` or ``hel1``

``AWS_S3_DOMAIN``
  base domain used to build endpoints such as ``amazonaws.com``

``AWS_S3_ENDPOINT_URL``
  string, default: ``https://{AWS_S3_REGION_NAME}.{AWS_S3_DOMAIN}``

  explicit endpoint, change if needed

``AWS_S3_CUSTOM_DOMAIN``
  string, default: ``{AWS_STORAGE_BUCKET_NAME}.{AWS_S3_REGION_NAME}.{AWS_S3_DOMAIN}``

  custom domain, change if needed

``USE_S3_URL_FOR_MEDIA``
  bool, default: ``True``

  when true, the app sets ``MEDIA_URL`` to
  ``https://{AWS_S3_CUSTOM_DOMAIN}/`` (otherwise use ``MEDIA_URL`` value). Set
  this to false only if you want to use a reverse proxy to make the media files
  appear under the same domain as the application, e.g. with nginx's ``proxy_pass``.

``USE_S3_URL_FOR_STATIC``
  bool, default: ``True``

  when true, the app sets ``STATIC_URL`` to
  ``https://{AWS_S3_CUSTOM_DOMAIN}/`` (otherwise use ``STATIC_URL`` value).

``S3_MEDIA_FILES_LOCATION`` and ``S3_STATIC_FILES_LOCATION``
  string, default: ``media`` and ``static``

  Used to control the prefix for media and static files in the bucket.

``AWS_QUERYSTRING_AUTH``
  bool, default: ``False``

  controls signed URLs; currently set to ``False`` (public URLs)

Examples for different providers:

**AWS**

* ``AWS_STORAGE_BUCKET_NAME=my-bucket``
* ``AWS_S3_REGION_NAME=eu-central-1``
* ``AWS_S3_DOMAIN=amazonaws.com``
* ``AWS_S3_ENDPOINT_URL=https://s3.eu-central-1.amazonaws.com``
* ``AWS_S3_CUSTOM_DOMAIN=my-bucket.s3.eu-central-1.amazonaws.com``

**Hetzner**

* ``AWS_STORAGE_BUCKET_NAME=my-bucket``
* ``AWS_S3_REGION_NAME=hel1``
* ``AWS_S3_DOMAIN=your-objectstorage.com``

For other options or services, consult the django-storages documentation:
https://django-storages.readthedocs.io

Note that iff you already have files in a local folder, you will need to
copy them over with a tool like ``awscli`` or ``rclone``.
