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

The full ingredient dataset takes around 1 GB of database space and needs a
while to download and process. For a quick start, load a smaller base set:

.. code-block:: bash

    docker compose exec web wger load-online-fixtures

For the full dataset:

.. code-block:: bash

    # On a fresh install:
    docker compose exec web ./manage.py sync-ingredients-bulk --set-mode insert

    # On an existing install (preserves your additions):
    docker compose exec web ./manage.py sync-ingredients-bulk --set-mode update

