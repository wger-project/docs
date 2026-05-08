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

Others
------


Building the image
~~~~~~~~~~~~~~~~~~

If you want to build your own image, you can do so by running the following
commands from the server's source folder:

.. code-block:: bash

    docker build -f extras/docker/development/Dockerfile -t wger/server .


There is also a "base" image located in ``extras/docker/base`` which the
server one uses as a base.

Next steps
----------

Once your installation is running, see the :ref:`administration` section for
ongoing operations:

* :doc:`/administration/lifecycle` — stop/start the application, run commands, set up auto-start with systemd
* :doc:`/administration/updating` — update wger to a new release
* :doc:`/administration/sync-data` — sync exercises and ingredients from upstream
* :doc:`/administration/backup` — back up and restore your database and media
* :doc:`/administration/postgres` — upgrade Postgres to a newer major version
* :doc:`/administration/monitoring` — monitor the application with Grafana and Prometheus
* :doc:`/administration/storage` — switch to SQLite or use S3-compatible object storage

