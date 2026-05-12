.. _backend:

Backend
===========

The backend is a Django application, the repository can be found at:

https://github.com/wger-project/wger

Development with docker
------------------------

:ref:`development_docker` provides a quick way to setup a development environment

Local installation
-------------------

Install ``npm``, ``sass`` and, optionally, `uv <https://docs.astral.sh/uv/>`_

Download the source code::

  git clone https://github.com/wger-project/wger.git server
  cd server

If using ``uv``::

    uv sync
    uv pip install -e .
    source .venv/bin/activate

Otherwise, manually create a new virtualenv and install everything::

  python3 -m venv .venv
  source .venv/bin/activate
  pip install --upgrade pip
  pip install --group dev .
  pip install -e .

This will download the required JS and CSS libraries and create an SQLite
database and populate it with data on the first run::

  export DJANGO_SETTINGS_MODULE=settings.local_dev
  export PYTHONPATH=/path/to/wger/server
  wger bootstrap
  wger load-online-fixtures

It's recommended to make a backup of the SQLite database after the initial
bootstrap, just copy it to some other place::

  cp database.sqlite database.sqlite.orig

You can of course also use other databases such as PostgreSQL or MariaDB. Create
a database and user and edit the DATABASES settings before calling bootstrap.
Take a look at the :ref:`PostgreSQL setup <prod_postgres>` for how that could look like.

Compile the translation files::

  cd wger
  django-admin compilemessages

After all this you can just use Django's development server::

  $ python manage.py runserver

That's it. You can log in with the default administrator user:

* **username**: admin
* **password**: adminadmin

You can reset the admin's password with ``wger create-or-reset-admin``.


And now
-------

* For a description of the available settings consult :ref:`settings`.

* You might need to start :ref:`celery` as well if you want to run certain
  commands in the background.

* For managing i18n files consult :ref:`i18n`.

* Check the :ref:`dummy_generator` for generating dummy data.

* Take a look at django extensions, a collection of custom extensions and
  commands:

  https://django-extensions.readthedocs.io/

  E.g. you can use ``runserver_plus`` instead of the default Django
  server as you can use an interactive debugger directly from the browser if an
  exception occurs. It also accepts the same command-line options. For this just
  install the following packages::

    pip install django_extensions werkzeug
    python manage.py runserver_plus [options]
