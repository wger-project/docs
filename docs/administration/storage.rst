.. _storage:

Storage
=======

This page covers the storage choices for your wger instance: which database
backend to use, and where to keep media and static files.

By default, wger uses Postgres for the database and stores media and static
files on local volumes (served by the bundled nginx). Both can be swapped out.

Database backend
----------------

The default Postgres setup is recommended for production. For small or test
installations, SQLite is also supported.

SQLite
~~~~~~

If you want to use an existing database, copy it next to the docker compose
file. Otherwise, create an empty file and make sure permissions are correct::

    touch ./database.sqlite
    chmod 664 ./database.sqlite

In your env file::

    DJANGO_DB_ENGINE=django.db.backends.sqlite3
    DJANGO_DB_DATABASE=/home/wger/db/database.sqlite

In ``docker-compose.yml``, mount the SQLite file into the web and celery
services, remove the dependency on the ``db`` service, and delete the ``db``
service definition entirely:

.. code-block:: yaml

    web:
      image: docker.io/wger/server:latest
      depends_on:
        # delete this
        db:
          condition: service_healthy
        [...]
      volumes:
        - ./database.sqlite:/home/wger/db/database.sqlite
        [...]

    celery_worker:
      image: docker.io/wger/server:latest
      volumes:
        - ./database.sqlite:/home/wger/db/database.sqlite
        [...]

    # remove the db service
    db:
      [...]

S3 / object storage
-------------------

wger supports serving static files (JS, CSS, etc.) and media files (gallery
images, exercise and ingredient images, etc.) from an S3-compatible object
store.

If you enable this, you don't need nginx to serve these files anymore — a
plain reverse proxy in front of the application is enough. You will likely
need to adjust the CORS settings on your storage provider so the application
can fetch the files from the S3 domain.

Settings
~~~~~~~~

``USE_S3_MEDIA_FILES``
  bool, default: ``False``

  Enable S3-backed media storage.

``USE_S3_STATIC_FILES``
  bool, default: ``False``

  Enable S3-backed static files storage. When enabled, set
  ``DJANGO_COLLECTSTATIC_ON_STARTUP`` to false — the collectstatic command
  will otherwise needlessly increase startup time.

``AWS_ACCESS_KEY_ID``
  access key

``AWS_SECRET_ACCESS_KEY``
  secret key

``AWS_STORAGE_BUCKET_NAME``
  bucket name

``AWS_S3_REGION_NAME``
  region used to build endpoints such as ``eu-central-1`` or ``hel1``

``AWS_S3_DOMAIN``
  base domain used to build endpoints such as ``amazonaws.com``

``AWS_S3_ENDPOINT_URL``
  string, default: ``https://{AWS_S3_REGION_NAME}.{AWS_S3_DOMAIN}``

  explicit endpoint, change if needed

``AWS_S3_CUSTOM_DOMAIN``
  string, default: ``{AWS_STORAGE_BUCKET_NAME}.{AWS_S3_REGION_NAME}.{AWS_S3_DOMAIN}``

  custom domain, change if needed

``USE_S3_URL_FOR_MEDIA``
  bool, default: ``True``

  When true, the app sets ``MEDIA_URL`` to ``https://{AWS_S3_CUSTOM_DOMAIN}/``
  (otherwise the configured ``MEDIA_URL`` value is used). Set this to false
  only if you want to use a reverse proxy to make the media files appear under
  the same domain as the application, e.g. with nginx's ``proxy_pass``.

``USE_S3_URL_FOR_STATIC``
  bool, default: ``True``

  When true, the app sets ``STATIC_URL`` to ``https://{AWS_S3_CUSTOM_DOMAIN}/``
  (otherwise the configured ``STATIC_URL`` value is used).

``S3_MEDIA_FILES_LOCATION`` and ``S3_STATIC_FILES_LOCATION``
  string, default: ``media`` and ``static``

  Used to control the prefix for media and static files in the bucket.

``AWS_QUERYSTRING_AUTH``
  bool, default: ``False``

  Controls signed URLs; currently set to ``False`` (public URLs).

Examples
~~~~~~~~

**AWS**

* ``AWS_STORAGE_BUCKET_NAME=my-bucket``
* ``AWS_S3_REGION_NAME=eu-central-1``
* ``AWS_S3_DOMAIN=amazonaws.com``
* ``AWS_S3_ENDPOINT_URL=https://s3.eu-central-1.amazonaws.com``
* ``AWS_S3_CUSTOM_DOMAIN=my-bucket.s3.eu-central-1.amazonaws.com``

**Hetzner**

* ``AWS_STORAGE_BUCKET_NAME=my-bucket``
* ``AWS_S3_REGION_NAME=hel1``
* ``AWS_S3_DOMAIN=your-objectstorage.com``

For other providers and options, consult the django-storages documentation:
https://django-storages.readthedocs.io

.. note::

   If you already have files in a local folder, you'll need to copy them
   over to the bucket with a tool like ``awscli`` or ``rclone``.
