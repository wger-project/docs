.. _installation:

Installation
============

You can get a local instance of wger installed in a couple of minutes.

It is recommended to install a development instance or start a docker
image if you just want to try the application on your server or PC. All
the following steps are performed on a Debian-based Linux distribution.
If your setup differs (e.g. in Red Hat based distros the package names are
slightly different) you will need to change the steps as appropriate.

You can safely install from master, it is almost always in a usable and
stable state.

The application is compatible and regularly tested with

* PostgreSQL, sqlite
* python 3.8, 3.9, 3.10 and pypy3

After installation, you might also want to take a look at the :ref:`other-changes` section for other
changes you might want to do to your local instance such as Terms of Service or
contact page.

.. note::
    Please note that all the steps related to upgrading the database or
    downloading external JS dependencies mentioned in the :ref:`tips` section
    in the development page apply to all the installation options.


These are the necessary packages for both development and production::

    sudo apt-get install nodejs npm git python3-dev python3-venv libcairo2-dev pkg-config
    sudo npm install -g yarn sass

If your node version is out of date, it can be updated with::

    sudo npm install -g n
    sudo npm cache clean -f
    sudo n stable

If you plan to use video files, you need to install additional packages, consult
the :ref:`videos` section for more information.

.. toctree::
   :maxdepth: 1

   development
   production
   celery
   docker
   updating

