.. _postgres_upgrade:

Postgres major version upgrade
==============================

Postgres does not automatically upgrade between major versions, you need to
perform the upgrade manually. Since the amount of data the application
generates is small, a simple dump-and-restore is the easiest approach.

If you pulled new changes from the docker-compose repository and got an error
message like *"The data directory was initialized by PostgreSQL version 12,
which is not compatible with this version 15"*, this page is for you.

See also: https://github.com/docker-library/postgres/issues/37

Procedure
---------

.. code-block:: bash

    # Check out the last version of the compose file that uses Postgres 12
    git checkout pg-12

    # Stop the other services
    docker compose stop web nginx cache celery_worker celery_beat

    # Dump the database, then remove the container and volume
    docker compose exec db pg_dumpall --clean --username wger > backup.sql
    docker compose stop db
    docker compose down
    docker volume remove docker_postgres-data

    # Switch back to the current version, import the dump, and start everything
    git checkout master
    docker compose up db
    cat backup.sql | docker compose exec -T db psql --username wger --dbname wger
    docker compose exec -T db psql --username wger --dbname wger -c "ALTER USER wger WITH PASSWORD 'wger'"
    docker compose up
    rm backup.sql

The ``pg-12`` tag in the docker-compose repository pins the previous Postgres
version. Adjust the tag if you're upgrading from a different version.

.. note::

   On a from-source installation, follow the standard Postgres upgrade
   procedure for your distribution (e.g., ``pg_upgradecluster`` on Debian/Ubuntu)
   or use ``pg_dumpall`` / ``pg_restore`` against your existing service.
