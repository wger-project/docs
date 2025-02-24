Production
==========

Wger user
---------

It is recommended to add a dedicated user for the application::

    sudo adduser wger --disabled-password --gecos ""

The following steps assume you did, but it is not necessary (nor is it
necessary to call it 'wger'). In that case, change the paths as needed.

Apache
------

Install apache and the WSGI module::

  sudo apt-get install apache2 libapache2-mod-wsgi-py3
  sudo vim /etc/apache2/sites-available/wger.conf


Configure apache to serve the application::

    <Directory /home/wger/src>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>


    <VirtualHost *:80>
        WSGIApplicationGroup %{GLOBAL}
        WSGIDaemonProcess wger python-path=/home/wger/src python-home=/home/wger/venv
        WSGIProcessGroup wger
        WSGIScriptAlias / /home/wger/src/wger/wsgi.py
        WSGIPassAuthorization On

        Alias /static/ /home/wger/static/
        <Directory /home/wger/static>
            Require all granted
        </Directory>

        Alias /media/ /home/wger/media/
        <Directory /home/wger/media>
            Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/wger-error.log
        CustomLog ${APACHE_LOG_DIR}/wger-access.log combined
    </VirtualHost>

Apache has a problem when uploading files that have non-ASCII characters, e.g.
for exercise images. To avoid this, add to /etc/apache2/envvars (if there is
already an ``export LANG``, replace it) or set your system's locale::

    export LANG='en_US.UTF-8'
    export LC_ALL='en_US.UTF-8'


Activate the settings and disable apache's default::

    sudo a2dissite 000-default.conf
    sudo a2ensite wger
    sudo service apache2 reload

Database
--------

.. _prod_postgres:

PostgreSQL
~~~~~~~~~~

Install the Postgres server (choose the appropriate and currently supported version
for your distro) and create a database and a user::

    sudo apt-get install postgresql postgresql-server-dev-12 python3-psycopg2
    sudo su - postgres
    createdb wger
    psql wger -c "CREATE USER wger WITH PASSWORD 'wger'";
    psql wger -c "GRANT ALL PRIVILEGES ON DATABASE wger to wger";

You might want or need to edit your ``pg_hba.conf`` file to allow local socket
connections or similar.


SQLite
~~~~~~

If using sqlite, create a folder for it (must be writable by the apache user)::

  mkdir db
  touch db/database.sqlite
  chown :www-data -R /home/wger/db
  chmod g+w /home/wger/db /home/wger/db/database.sqlite

Application
-----------

As the wger user, make a virtualenv for python and activate it::

  python3 -m venv /home/wger/venv
  source /home/wger/venv/bin/activate

Create folders to collect all static resources and save uploaded files. The
``static`` folder will only contain CSS and JS files, so it must be readable
by the apache process while ``media`` will contain the uploaded files and must
be writeable as well::

  mkdir static

  mkdir media
  chmod o+w media

Get the application::

  git clone https://github.com/wger-project/wger.git /home/wger/src
  cd /home/wger/src
  pip install -r requirements.txt
  pip install -e .

  # If using sqlite without the --database-path
  wger create-settings --database-path /home/wger/db/database.sqlite

Edit the settings file

* Add the correct values for the database (use ``django.db.backends.postgresql``
  for the engine) if you are using postgres

* Set ``MEDIA_ROOT`` to ``/home/wger/media`` and ``STATIC_ROOT`` to ``/home/wger/static``.

* Add the domains that your site will be accessed to `ALLOWED_HOSTS=['example.com', 'www.example.com']`
  (you might want to do this as the last step when you know everything else is
  working correctly)

Run the installation script, this will download some CSS and JS libraries and
load all initial data::

  wger bootstrap


Collect all static resources::

  python manage.py collectstatic


The bootstrap command will also create a default administrator user (you probably
want to change the password as soon as you log in):


* **username**: admin
* **password**: adminadmin

.. _email:

Email
-----

The application is configured to use Django's console email backend by default, which causes messages intended to be sent via email to be written to ``stdout``.

In order to use a real email server, another backend listed in `Django's documentation`_ can be configured instead. Parameters for the backend are set as variables in ``settings.py``. For example, the following allows an SMTP server at ``smtp.example.com`` to be used::

   Email_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
   ENABLE_EMAIL = True
   EMAIL_HOST = 'smtp.example.com'
   EMAIL_PORT = 587
   EMAIL_HOST_USER = 'wger@example.com'
   EMAIL_HOST_PASSWORD = 'example_password'
   EMAIL_USE_TLS = True
   EMAIL_USE_SSL = False
   DEFAULT_FROM_EMAIL = 'wger Workout Manager <wger@example.com>'

Django provides a ``sendtestemail`` command via ``manage.py`` to test email settings::

  python manage.py sendtestemail user@example.com

.. _`Django's documentation`: https://docs.djangoproject.com/en/dev/topics/email/#email-backends

.. _site-settings:

Site Settings
-------------

Some wger features make use of Django's site name and domain settings in the ``contrib.sites`` framework. These should be set through the Python shell::

   python manage.py shell
   >>> from django.contrib.sites.models import Site
   >>> site = Site.objects.get(pk=1)
   >>> site.domain = 'wger.example.com'
   >>> site.name = 'example.com wger Workout Manager'
   >>> site.save()

where ``wger.example.com`` is the domain of the wger instance. This assumes that wger is using the default site ID of 1. If a different site ID is being used, it must be specified in ``settings.py``::

  SITE_ID = 2

.. _other-changes:

Other changes
-------------

If you want to use the application as a public instance, you will probably want to
change the following templates:

* **tos.html**, for your own Terms Of Service here
* **about.html**, for your contact address or other such legal requirements
