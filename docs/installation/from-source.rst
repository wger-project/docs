.. _installation:

Installation from source
========================

Install ``npm`` / ``nodejs`` >= 22,  and ``sass``

If you plan to use video files, it's recommended you install ffmpeg as well
(this is not strictly necessary, it's just used for better upload
validation as well as the extraction of some information from the video files
such as resolution, codec, etc)::

    apt-get install ffmpeg
    pip install ffmpeg-python

wger user
---------

It is recommended to add a dedicated user for the application::

    sudo adduser wger --disabled-password --gecos ""

The following steps assume you did, but it is not necessary (nor is it
necessary to call it 'wger'). In that case, change the paths as needed.


Database
--------

.. _prod_postgres:

**PostgreSQL**

Install the Postgres server, then create a database and a user::

    sudo apt-get install postgresql
    sudo su - postgres
    createdb wger
    psql wger -c "CREATE USER wger WITH PASSWORD 'wger'";
    psql wger -c "GRANT ALL PRIVILEGES ON DATABASE wger to wger";

The ``postgresql`` meta-package pulls in whatever currently-supported version
your distro packages, any Postgres ≥ 15 should work.

You might need to edit your ``pg_hba.conf`` to allow local socket connections.


**SQLite**

If using sqlite, create a folder for it (must be writable by the wger user)::

  mkdir /home/wger/db
  touch /home/wger/db/database.sqlite
  chown :www-data -R /home/wger/db
  chmod g+w /home/wger/db /home/wger/db/database.sqlite

Application
-----------

Install `uv <https://docs.astral.sh/uv/>`_ if you don't have it already, then
clone the repository and install wger with the ``docker`` dependency group.
This pulls in gunicorn alongside the base dependencies (for the systemd unit
we'll create in the Webserver section below)::

  git clone https://github.com/wger-project/wger.git /home/wger/src
  cd /home/wger/src
  uv sync --group docker --no-managed-python

The ``--no-managed-python`` flag tells uv to use your system's Python
(installed via apt or similar) instead of downloading its own. Drop the flag
if you'd rather have uv manage it.

uv creates the virtualenv at ``/home/wger/src/.venv`` automatically. Activate
it for the bootstrap commands below::

  source /home/wger/src/.venv/bin/activate

Create folders for static resources and uploaded files. The ``static`` folder
contains generated CSS and JS files (read by Caddy), the ``media`` folder
receives uploads (read by Caddy, written by gunicorn/django)::

  mkdir /home/wger/{static,media}
  chmod o+w /home/wger/media

**Configuration**

wger reads its configuration from environment variables. Place them in
``/home/wger/wger.env`` so the systemd unit we'll define in the Webserver
section can pick them up (via its ``EnvironmentFile=`` directive), and so
you can source them into a shell when running one-off commands.

For a complete commented list of available settings, use the Docker
setup's ``prod.env`` as a reference:
https://github.com/wger-project/docker/blob/master/config/prod.env

A minimal example:

.. code-block:: bash

    # /home/wger/wger.env

    # Change these
    ALLOWED_HOSTS=example.com,www.example.com
    DJANGO_SECRET_KEY=your-very-long-and-random-secret-key
    TIME_ZONE=Europe/Berlin

    # Application
    MEDIA_ROOT=/home/wger/media
    STATIC_ROOT=/home/wger/static

    # Django Setup
    DJANGO_SETTINGS_MODULE=settings.main
    PYTHONPATH=/home/wger/src

    # Postgres
    DJANGO_DB_ENGINE=django.db.backends.postgresql
    DJANGO_DB_NAME=wger
    DJANGO_DB_USER=wger
    DJANGO_DB_PASSWORD=wger
    DJANGO_DB_HOST=localhost
    DJANGO_DB_PORT=5432

For SQLite, swap the database block for::

    DJANGO_DB_ENGINE=django.db.backends.sqlite3
    DJANGO_DB_NAME=/home/wger/db/database.sqlite

The file contains secrets, so restrict permissions::

    sudo chown wger:wger /home/wger/wger.env
    sudo chmod 600 /home/wger/wger.env

.. note::

   The file uses systemd's ``EnvironmentFile`` syntax: ``KEY=VALUE`` per
   line, no ``export`` prefix and no spaces around ``=``. Quotes around
   values are usually unnecessary; if you do need them, use single quotes.

**Bootstrap**

For one-off commands (bootstrap, migrate, collectstatic, …) the env file
needs to be sourced into the current shell. As the wger user with the
virtualenv active::

    set -a; . /home/wger/wger.env; set +a

Run the installation script, this downloads JS/CSS libraries and loads
initial data::

    wger bootstrap

Collect all static resources::

    python manage.py collectstatic

Compile the translation (.po) files::

    cd wger
    django-admin compilemessages

The bootstrap command also creates a default administrator user. **Change
the password immediately after first login:**

* **username**: admin
* **password**: adminadmin


Webserver
---------

The recommended setup runs **gunicorn** as the WSGI application server, with
**Caddy** as a reverse proxy in front of it that also serves the static and
media files. This mirrors how the Docker image is configured internally.

If you prefer nginx, Apache or another webserver, the building blocks are the
same, just set up a reverse proxy that forwards to gunicorn and serves
``/static/`` and ``/media/`` directly. See Django's deployment guide:
https://docs.djangoproject.com/en/dev/howto/deployment/

**gunicorn**

Create a systemd service file at ``/etc/systemd/system/wger.service``:

.. code-block:: ini

    [Unit]
    Description=wger gunicorn daemon
    After=network.target

    [Service]
    User=wger
    Group=wger
    WorkingDirectory=/home/wger/src
    EnvironmentFile=/home/wger/wger.env
    ExecStart=/home/wger/src/.venv/bin/gunicorn wger.wsgi:application \
        --preload \
        --bind 127.0.0.1:8000 \
        --workers 3 \
        --threads 2 \
        --worker-class gthread \
        --timeout 240 \
        --access-logfile -

    [Install]
    WantedBy=multi-user.target

The worker count of ``(2 × $num_cores) + 1`` is a common starting point. The
unit reads your env vars from ``/home/wger/wger.env`` via ``EnvironmentFile``.

Reload systemd and start the service::

    sudo systemctl daemon-reload
    sudo systemctl enable --now wger

Check that it's running with ``systemctl status wger`` and
``journalctl -u wger -f``.

**Caddy**

Install Caddy following the official instructions:
https://caddyserver.com/docs/install

Create ``/etc/caddy/Caddyfile``:

.. code-block::

    your-domain.example.com {
        encode

        reverse_proxy 127.0.0.1:8000 {
            header_up Host {host}
            header_up X-Real-IP {remote_host}
            header_up X-Forwarded-For {http.X-Forwarded-For} {remote_host}
            header_up X-Forwarded-Proto {scheme}
            header_up X-Http-Version {http.request.proto}
        }

        handle /static/* {
            root * /home/wger
            header Cache-Control "public, max-age=31536000, immutable"
            file_server
        }

        handle /media/* {
            root * /home/wger
            file_server
        }
    }

Caddy automatically obtains and renews a Let's Encrypt certificate as long as
the domain's DNS points to your server. Reload Caddy to pick up the config::

    sudo systemctl reload caddy

The ``X-Forwarded-*`` headers are important, without them, generated URLs
(e.g. pagination links in the API) and CSRF protection can break. If you see
CSRF errors after setup, check :doc:`/administration/errors`.

.. note::

   If ``/static/`` or ``/media/`` return 403 or 404, check whether the webserver's
   systemd unit has ``ProtectHome=true``, as it blocks reads from ``/home/wger/``.
   Override the unit::

       sudo systemctl edit caddy

   and add::

       [Service]
       ProtectHome=off

   Then reload: ``sudo systemctl daemon-reload && sudo systemctl restart caddy``.


Next steps
----------

Once your installation is running, see the :ref:`administration` section for
ongoing operations.
