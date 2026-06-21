.. _powersync:

PowerSync
=========

PowerSync is the service that gives the mobile app bidirectional, offline-capable
sync. It replicates from the application's Postgres database into *its own* bucket
storage and exposes a sync endpoint the app connects to. See :ref:`architecture`
for how it fits into the overall system.

.. _powersync_setup:

Initial setup
-------------

PowerSync keeps its sync state in a dedicated Postgres role and a separate schema
inside the wger database. These have to be created **once** before the service
can work and is  **not** done automatically.

Run::

    docker compose exec web ./manage.py setup-powersync-storage

The command reads the connection details from ``PS_STORAGE_PG_URI`` and creates
the role and schema if they are missing. It is idempotent and safe to re-run, on
an existing role it simply updates the password to match the URI.

If you change the storage password, update ``PS_STORAGE_PG_URI`` first and then
re-run the command. Pass ``--schema`` to use a schema name other than the
default ``powersync``.


.. _powersync_maintenance:

Maintenance
-----------

The service replicates every ``INSERT``/``UPDATE``/``DELETE`` on a synced table
into its bucket storage as an append-only operation log. Without periodic
maintenance those logs grow unbounded.

Compaction runs online, clients can keep syncing while it is in progress.

When to run it
~~~~~~~~~~~~~~

PowerSync's self-hosted service does **not** ship with a built-in scheduler.
You have to invoke compaction yourself, typically from cron. A daily run is
a reasonable default; instances with very high write volume on workout logs
or nutrition logs can run it hourly.

Running compaction
~~~~~~~~~~~~~~~~~~

Run the ``compact`` subcommand in a one-off PowerSync container:

.. code-block:: bash

    docker compose run --rm powersync compact

Scheduling with cron
~~~~~~~~~~~~~~~~~~~~

.. code-block::

    # /etc/cron.d/wger-powersync-compact
    0 3 * * *  root  cd /path/to/wger/docker && docker compose run --rm -T powersync compact >>/var/log/wger-compact.log 2>&1

The ``-T`` flag disables TTY allocation, which is required when running
from cron.

Scheduling with systemd
~~~~~~~~~~~~~~~~~~~~~~~

If you already use systemd to manage wger (see :ref:`lifecycle`), a
timer pairs naturally with it. Two unit files are needed.

The service unit, ``/etc/systemd/system/wger-powersync-compact.service``:

.. code-block:: ini

    [Unit]
    Description=wger PowerSync bucket compaction
    Requires=docker.service wger.service
    After=docker.service wger.service

    [Service]
    Type=oneshot
    WorkingDirectory=/path/to/wger/docker
    ExecStart=/usr/bin/docker compose run --rm -T powersync compact

The timer unit, ``/etc/systemd/system/wger-powersync-compact.timer``:

.. code-block:: ini

    [Unit]
    Description=Run wger PowerSync compaction daily

    [Timer]
    OnCalendar=*-*-* 03:00:00
    Persistent=true
    RandomizedDelaySec=15min

    [Install]
    WantedBy=timers.target

Enable and start the timer:

.. code-block:: bash

    systemctl daemon-reload
    systemctl enable --now wger-powersync-compact.timer

Verify with:

.. code-block:: bash

    systemctl list-timers wger-powersync-compact.timer
    journalctl -u wger-powersync-compact.service

``Persistent=true`` catches up on a missed run if the host was down at
03:00; ``RandomizedDelaySec`` avoids hammering the DB at exactly the
same wall-clock time as other scheduled jobs.

Defaults you may want to know:

* ``minBucketChanges`` (default 10): minimum new ops before a bucket is
  considered for compaction.
* ``minChangeRatio`` (default 0.1): minimum ratio of new to existing ops.

These prevent wasted work on essentially-idle buckets.