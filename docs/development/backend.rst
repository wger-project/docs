.. _backend:

Backend
=======

The backend is a Django application, the repository can be found at:

https://github.com/wger-project/wger

.. tip::

   The easiest way to get a backend running is via the Docker dev environment,
   see :ref:`development_docker`. The instructions below cover a native install
   on your machine.

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
  wger bootstrap
  wger load-online-fixtures

It's recommended to make a backup of the SQLite database after the initial
bootstrap, just copy it to some other place::

  cp database.sqlite database.sqlite.orig

You can of course also use other databases such as PostgreSQL or MariaDB. Create
a database and user with the usual tools (``createdb``, ``CREATE USER``, etc.)
and point Django at it via the ``DJANGO_DB_*`` env vars (see :doc:`/administration/settings`)
before calling bootstrap.

Compile the translation files::

  cd wger
  django-admin compilemessages

After all this you can just use Django's development server::

  $ python manage.py runserver

That's it. You can log in with the default administrator user:

* **username**: admin
* **password**: adminadmin

You can reset the admin's password with ``wger create-or-reset-admin``.


Running the tests
-----------------

The backend uses Django's built-in test runner. Each app keeps its tests under
``wger/<app>/tests/``. To run the tests::

  # Full text suite
  python manage.py test

  # In multi core machines
  python manage.py test --parallel auto

  # Only some tests
  python manage.py test wger.exercises
  python manage.py test wger.exercises.tests.test_categories


Code style
----------

The project uses ``ruff`` for formatting and ``isort`` for import sorting, both
configured in ``pyproject.toml``. Before opening a PR::

  ruff format
  isort .

CI runs the same checks, so it's worth doing it locally first.


Next steps
----------

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

