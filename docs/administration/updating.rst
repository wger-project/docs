.. _updating:

Updating wger
=============

Pull new releases regularly to get bug fixes, new features and security
updates.

**Docker**

Remove the containers and pull the newest images:

.. code-block:: bash

    docker compose down
    docker compose pull
    docker compose up -d

That's it, database migrations and static files are applied automatically on
container start.

**From source**

Pull new changes and apply them in the source tree:

.. code-block:: bash

    git pull
    pip install .
    python manage.py migrate --all
    npm install
    npm run build:css:sass
    python manage.py collectstatic

If you've configured Celery for periodic data sync, the exercise and ingredient
datasets stay current automatically. Otherwise, see :doc:`sync-data` for the
manual sync commands.
