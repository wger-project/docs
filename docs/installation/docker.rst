.. _docker_prod:

Docker compose
==============

https://github.com/wger-project/docker

The prod docker compose file starts up gunicorn as the application server,
postgres as the database, redis for caching, and nginx as a reverse proxy.

The database, static files and uploaded images are mounted as volumes so
the data is persisted. The only thing you need to do is update the docker
images. Consult the docker volume command for details on how to access or
backup this data.

It is recommended to regularly pull the latest version of the compose file,
since sometimes new configurations or environmental variables are added.

If after installing not everything works, consult the :ref:`errors_and_pitfalls`
section for common errors and how to fix them.


Quickstart
----------
To start all services::

    docker compose up -d

Then open http://localhost (or your server's IP) and log in as: **admin**,
password **adminadmin**.

.. warning::
    If your instance is reachable over the internet, change the default password
    after logging in for the first time and change the JWT keys.

Be sure to read ``config/prod.env``and at least set these settings:

* ``SECRET_KEY``with ``python -c "import secrets; print(secrets.token_urlsafe(50))"``
* ``JWT_PRIVATE_KEY`` and ``JWT_PUBLIC_KEY`` with ``docker compose exec web ./manage.py generate-jwt-keys``
* ``SITE_URL`` with your server's URL


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

The docker repo ships a `Caddyfile.example
<https://github.com/wger-project/docker/blob/master/config/Caddyfile.example>`_
for users who'd rather use Caddy than nginx in front of the application. Caddy
automatically obtains and renews SSL certificates from Let's Encrypt.

Also notice that the application currently needs to run on its own (sub)domain
and not in a subdirectory, so ``<domain>/wger`` will probably only mostly work.

Next steps
----------

Once your installation is running, see the :ref:`administration` section for
ongoing operations (specially the one about powersync).