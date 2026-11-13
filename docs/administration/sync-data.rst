.. _sync_data:

Syncing exercises and ingredients
=================================

The Docker image ships with a default set of exercises but no ingredients
(the dataset is large). You can manually sync both from wger.de or any other
instance using these commands.

The application can also be configured to perform these steps automatically
in the background via Celery; see the ``SYNC_*`` options in :doc:`settings`.

.. note::

   Sync commands never overwrite exercises or ingredients you've added
   yourself to your instance.

Exercises
---------

.. code-block:: bash

    docker compose exec web python3 manage.py sync-exercises
    docker compose exec web python3 manage.py download-exercise-images
    docker compose exec web python3 manage.py download-exercise-videos

Ingredients
-----------

A new installation has no ingredients. Load them once with one of the two
options below; afterwards they can be kept up to date automatically.

Option 1: base set
~~~~~~~~~~~~~~~~~~

About 177,000 ingredients (a 30 MB download). Good for a quick start, but it
contains no serving units (e.g. "1 slice") and the data is not current.

.. code-block:: bash

    docker compose exec web wger load-online-fixtures

.. warning::

   Only load the base set into an instance **without ingredients**. The file
   contains fixed IDs and overwrites existing ingredients with the same ID.

You can switch to the full dataset later with option 2 in ``update`` mode.

Option 2: full dataset
~~~~~~~~~~~~~~~~~~~~~~

All ingredients from wger.de, including serving units. Needs around 1 GB of
database space and takes a while to download and process.

.. code-block:: bash

    # Instance without ingredients:
    docker compose exec web ./manage.py sync-ingredients-bulk --set-mode insert

    # Instance with ingredients, e.g. after loading the base set
    # (preserves your additions):
    docker compose exec web ./manage.py sync-ingredients-bulk --set-mode update

Keeping them up to date
~~~~~~~~~~~~~~~~~~~~~~~

With ``SYNC_INGREDIENTS_CELERY=True`` (the default in the Docker setup) the
full dataset is synced in ``update`` mode once a month, on a random day. The
first run can be up to a month after the installation, so load the
ingredients manually first.

Images are not part of either option. With ``DOWNLOAD_INGREDIENTS_FROM=WGER``
and Celery they are downloaded in the background the first time an
ingredient is opened. To download all images at once:

.. code-block:: bash

    docker compose exec web ./manage.py download-ingredient-images

