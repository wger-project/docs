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

Other setups
------------

  **Kubernetes**
    There is a helm charts repository if you want to deploy the application with
    kubernetes: https://github.com/wger-project/helm-charts

  **TrueNAS SCALE**
    Consult this guide if you want to deploy the application to :ref:`truenas`

First steps
-----------
To start all services::

    docker compose up -d

Optionally download current exercises, exercise images and the ingredients
from wger.de. Please note that ``load-online-fixtures`` will overwrite any local
changes you might have while ``sync-ingredients`` should be used afterward once
you have imported the initial fixtures:

.. code-block:: bash

    docker compose exec web python3 manage.py sync-exercises
    docker compose exec web python3 manage.py download-exercise-images
    docker compose exec web python3 manage.py download-exercise-videos

    # Loads a base set of ingredients
    docker compose exec web wger load-online-fixtures

    # optionally run this afterwards to sync all the ingredients (around 1GB,
    # this process takes a loooong time):
    docker compose exec web python3 manage.py sync-ingredients-async

(these steps are configured by default to run regularly in the background, but
can also run on startup as well, see the options in ``prod.env``.)


Then open http://localhost (or your server's IP) and log in as: **admin**,
password **adminadmin**

**Update the application**

Just remove the containers and pull the newest version:

.. code-block:: bash
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

Others / Common error and pitfalls
----------------------------------

CSRF errors
```````````

You will most probably run into CSRF errors when you try to use the application,
specially if you configured a domain and django's
`CSRF protection <https://docs.djangoproject.com/en/dev/ref/csrf/>`_ kicks in.
To solve this, update the env file and either

* manually set a list of your domain names and/or server IPs
  ``CSRF_TRUSTED_ORIGINS=https://my.domain.example.com,https://118.999.881.119:8008``
  If you are unsure what origin to add here, set set ``DJANGO_DEBUG`` to true,
  restart the service and the error message will tell you exactly which one
  django has a problem with. Note: the port is important!
* or set the ``X-Forwarded-Proto`` header like in the example and set
  ``X_FORWARDED_PROTO_HEADER_SET=True``. If you do this consult the
  `documentation <https://docs.djangoproject.com/en/4.1/ref/settings/#secure-proxy-ssl-header>`_
  as there are some security considerations.

Missing static files
````````````````````

If you start the application and don't see any CSS styles, images, etc., there's a
problem with the static files. This happens often.

The reason for this is that for performance reasons, django itself does not serve
any of the static files in production. Instead, it relies on a separate dedicated
process server (in our case, the nginx service, but it could be an external CDN
or similar) to do it. The process works in two steps:

* **Collect:** the django service runs a command to gather all static files into a
  single directory. This step happens automatically when you start the ``web``
  service but can be manually triggered with ``docker compose exec web python3
  manage.py collectstatic``.
* **Serve:** the nginx service reads files from that exact same directory and
  serves them to the user.

These two steps need access to shared docker volume. If you change the volume
configuration, django might not be able to write the file or the web server might
no longer find them. This can happen very easily if you use mounted folders and
the permissions aren't correctly set (make sure they are ``chown``-ed to the UID
and GID 1000, even if this user doesn't exist on your system, and are readable by
everyone).

The solution is always to ensure that the volume or folder which holds your
static files is correctly mounted by both the django and the web server services.
If you want to use your own, exising, web server, you need to make sure that
the files are read and served under the right URLS. Take a look at our nginx.conf
to see how this can look like.

For more information, consult `django's documentation <https://docs.djangoproject.com/en/4.1/ref/settings/#secure-proxy-ssl-header>`_.



Automatically start service
```````````````````````````

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
``````

**Database volume:** The most important thing to backup. For this just make
a dump and restore it when needed

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

**Media volume:** If you haven't uploaded any own images (exercises, gallery),
you don't need to backup this, the contents can just be downloaded again. If
you have, please consult these possibilities:

* https://www.docker.com/blog/back-up-and-share-docker-volumes-with-this-extension/
* https://github.com/BretFisher/docker-vackup


**Static volume:** The contents of this volume are 100% generated and recreated
on startup, no need to backup anything

Postgres Upgrade
````````````````

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
``````````````````

If you want to build your own image, you can do so by running the following
commands from the server's source folder:

.. code-block:: bash

    docker build -f extras/docker/development/Dockerfile -t wger/server .


There is also a "base" image located in ``extras/docker/base`` which the
server one uses as a base.
