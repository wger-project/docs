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

Install ``npm``, ``yarn`` (1.x) ``sass`` and, optionally, `uv <https://docs.astral.sh/uv/>`_

Download the source code::

  git clone https://github.com/wger-project/wger.git server
  cd server

If using ``uv``::

    uv sync
    uv pip install -e .
    source .venv/bin/activate

Overwise, manually create a new virtualenv and install everything::

  python3 -m venv .venv
  source .venv/bin/activate
  pip install --upgrade pip
  pip install --group dev .
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


And now
-------

* For a description of the available settings consult :ref:`settings`.

* You might need to start :ref:`celery` as well if you want to to run certain
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


Release process
---------------
Before releasing a new (major or minor) version of the backend, a couple of manual
steps are still necessary.

Yes, some of these steps could and should be automated, but for now they are not.
This might also be one of the reasons why there are not that many releases...

1) Bump version

Bump the version in:

* ``wger/version.py``
* ``package.json`` (not really needed, but since it's there, let's keep it up to date)
* All the ``.github/workflows/docker-*.yml`` files
* ``docs/conf.py`` (in the docs repo)


2) Update contributors list

Run the script that updates the contributors list::

  python3 extras/authors/generate_authors_api.py


3) Update exercise fixture

It's recommended to update the exercise fixture before a release. To do this extract
them from a current database, split the files and and copy them as appropriate::

    python ./manage.py dumpdata --indent 4 --natural-foreign exercises > extras/scripts/data.json
    cd extras/scripts/
    python3 filter-fixtures.py
    cp categories.json ../../wger/exercises/fixtures/

4) Update translations

Update the po files as described in :ref:`i18n`.

5) Tag the release

Create a new tag for the release::

  git tag -a 1.2.3 -m "Release 1.2.3"
  git push origin 1.2.3

6) Create a new release on GitHub

Finally, create a new release on GitHub from the tag. Generate the description
from the pull requests and edit if necessary. Copy this changelog to the docs
repo and add it to the existing changelog.rst.

7) Talk about it!

Write an announcement, and post it on discord, mastodon, etc.
