.. _updating:

Updating wger
=============

Pull new releases regularly to get bug fixes, new features and security
updates.

**Every release has a page at https://github.com/wger-project/wger/releases**
with what changed and with anything that particular update needs beyond the
steps below. Read it before you start, it is where the exceptions are spelled
out.

**Docker**

Remove the containers and pull the newest images:

.. code-block:: bash

    docker compose down
    docker compose pull
    docker compose up -d

That's it, database migrations and static files are applied automatically on
container start.

.. warning::
    The PowerSync sync rules are **not** part of the images: they are bind
    mounted from a folder in the docker repository, so new images leave your
    copy untouched. Pull that repository as well and restart the service:

    .. code-block:: bash

        git pull
        docker compose restart powersync

    Nothing warns you if you skip it. The service starts and the logs stay
    clean, while the app quietly misses data. See :ref:`powersync`.

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
