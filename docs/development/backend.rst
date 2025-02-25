.. _development:

Backend
===========

Development with docker
------------------------

:ref:`development_docker` provides a quick way to get a development environment

Local installation
-------------------

Install ``npm``, ``yarn`` (1.x) and ``sass``

Create a new virtualenv::

  python3 -m venv venv-wger
  source venv-wger/bin/activate

Download the source code::

  git clone https://github.com/wger-project/wger.git src
  cd src

Install python requirements::

  pip install -r requirements_dev.txt
  pip install -e .

This will download the required JS and CSS libraries and create an SQLite
database and populate it with data on the first run::

  wger create-settings
  wger bootstrap
  wger load-online-fixtures

It's recommended to make a backup of the SQLite database after the initial
bootstrap, just copy it to some other place::

  cp database.sqlite database.sqlite.orig

You can of course also use other databases such as PostgresSQL or MariaDB. Create
a database and user and edit the DATABASES settings before calling bootstrap.
Take a look at the :ref:`prod_postgres` on apache on how that could look like.

After all this you can just use Django's development server::

  $ python manage.py runserver

That's it. You can log in with the default administrator user:

* **username**: admin
* **password**: adminadmin

You can reset the admin's password with ``wger create-or-reset-admin``.


Other
-----
* For a description of the available settings consult :ref:`settings`.

* You might need to start :ref:`celery` as well if you want to to run certain
  commands in the background.

* For managing i18n files consult :ref:`i18n`.

* Take a look at django extensions, a collection of custom extensions and
  commands:

  https://django-extensions.readthedocs.io/

  E.g. you can use ``runserver_plus`` instead of the default Django
  server as you can use an interactive debugger directly from the browser if an
  exception occurs. It also accepts the same command-line options. For this just
  install the following packages::

    pip install django_extensions werkzeug
    python manage.py runserver_plus [options]
