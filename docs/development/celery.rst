.. _celery:

Celery
------

wger uses a celery queue for some background tasks. At the moment this is used
for things like fetching the ingredient images or periodically synchronizing the
exercise database.

The celery queue is optional and is not required.

Celery needs a configured cache backend, redis is a good and easy solution and
you might already have it running for the regular application::

    CELERY_BROKER_URL = "redis://localhost:6379/2"
    CELERY_RESULT_BACKEND = "redis://localhost:6379/2"

Finally, set the option in settings.py to use it::

    WGER_SETTINGS['USE_CELERY'] = True

For alternatives, consult celery's documentation: https://docs.celeryq.dev/en/stable/

.. note::
  The docker compose file has all services already configured, you don't need
  to do anything. These installation instructions are only for a manual setup.



Celery worker
=============

For development, to start the celery worker just run in your virtual env::

    celery -A wger worker -l INFO


In a production setting you will need to properly daemonize the service. One
option is a systemd service file. If you are using the structure and names used
in the production chapter of this documentation, you can proceed as follows.

Create a folder in where we will put the configuration::

    mkdir /home/wger/celery/

Create a configuration file called ``conf`` and paste this::

    # Name of nodes to start
    CELERYD_NODES="w1"

    # Absolute path to the 'celery' command within the venv
    CELERY_BIN="/home/wger/venv/bin/celery"

    # App instance to use
    CELERY_APP="wger"

    # How to call manage.py
    CELERYD_MULTI="multi"

    # Extra command-line arguments to the worker
    CELERYD_OPTS="--time-limit=300 --concurrency=8"

    # - %n will be replaced with the first part of the nodename.
    # - %I will be replaced with the current child process index
    #   and is important when using the prefork pool to avoid race conditions.
    CELERYD_PID_FILE="/var/run/celery/%n.pid"
    CELERYD_LOG_FILE="/var/log/celery/%n%I.log"
    CELERYD_LOG_LEVEL="INFO"

    CELERYBEAT_PID_FILE="/var/run/celery/beat.pid"
    CELERYBEAT_LOG_FILE="/var/log/celery/beat.log"

Since we're running this as the wger user, we need to make sure the process can
write the logs and the pid files. For this edit a new file ``/etc/tmpfiles.d/celery.conf``,
past the following content and reboot the system::

    d /run/celery 0755 wger wger -
    d /var/log/celery 0755 wger wger -


Add a new file `/etc/systemd/system/celery.service` with the following contents
(if you don't have a redis service or is called differently, adjust as needed):

.. code-block:: ini

    [Unit]
    Description=Celery Service
    After=network.target
    After=redis.service
    Requires=redis.service

    [Service]
    Type=forking
    User=wger
    Group=wger
    EnvironmentFile=/home/wger/celery/conf
    WorkingDirectory=/home/wger/src
    ExecStart=/bin/sh -c '${CELERY_BIN} -A $CELERY_APP multi start $CELERYD_NODES \
        --pidfile=${CELERYD_PID_FILE} \
        --logfile=${CELERYD_LOG_FILE} \
        --loglevel="${CELERYD_LOG_LEVEL}" $CELERYD_OPTS'
    ExecStop=/bin/sh -c '${CELERY_BIN} multi stopwait $CELERYD_NODES \
        --pidfile=${CELERYD_PID_FILE} \
        --logfile=${CELERYD_LOG_FILE} \
        --loglevel="${CELERYD_LOG_LEVEL}"'
    ExecReload=/bin/sh -c '${CELERY_BIN} -A $CELERY_APP multi restart $CELERYD_NODES \
        --pidfile=${CELERYD_PID_FILE} \
        --logfile=${CELERYD_LOG_FILE} \
        --loglevel="${CELERYD_LOG_LEVEL}" $CELERYD_OPTS'
    Restart=always

    [Install]
    WantedBy=multi-user.target

Read the file with ``systemctl daemon-reload`` and start it with ``systemctl start celery``.
If there are no errors and ``systemctl status celery`` shows that the service is
active, everything went well. With ``systemctl enable celery.service``the service
will be automatically restarted after a reboot.

For more up to date information on how this could look like:
https://docs.celeryq.dev/en/stable/userguide/daemonizing.html



Celery beat
===========

Celery beat is used to perform periodic tasks. This is used at the moment to
regularly sync the exercises from the configured wger instance. A random time
and day of the week is selected in which the individual task will be run. Each
task can be toggled on or off with a setting in the ``WGER_SETTING`` dictionary:

* **SYNC_EXERCISES_CELERY** to synchronize the exercises themselves
* **SYNC_EXERCISE_IMAGES_CELERY** to synchronize exercise images
* **SYNC_EXERCISE_VIDEOS_CELERY** to synchronize exercise videos

To start it just run in your virtual env::

    celery -A wger beat -l INFO

To daemonize this you just need to add a new service, e.g.
``/etc/systemd/system/celery-beat.service``:

.. code-block:: ini

    [Unit]
    Description=Celery Beat Service
    After=network.target
    After=celery.service
    Requires=celery.service

    [Service]
    Type=forking
    User=wger
    Group=wger
    EnvironmentFile=/home/wger/celery-conf/celery
    WorkingDirectory=/home/wger/src
    ExecStart=/bin/sh -c '${CELERY_BIN} -A ${CELERY_APP} beat \
        --pidfile=${CELERYBEAT_PID_FILE} \
        --logfile=${CELERYBEAT_LOG_FILE} \
        --loglevel=${CELERYD_LOG_LEVEL}'
    Restart=always

    [Install]
    WantedBy=multi-user.target


Then as above, reload the server and start the service::

    systemctl daemon-reload
    systemctl start celery-beat

Celery flower
=============

Celery flower is a web app that allows you to take a look at the performed tasks

To start it just run in your virtual env::

    celery -A wger --broker="${CELERY_BROKER}" flower

