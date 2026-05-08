.. _lifecycle:

Lifecycle
=========

Day-to-day commands for stopping, starting and inspecting your wger
installation.

Basic commands
--------------

To stop all services while preserving containers and volumes::

    docker compose stop

To start everything up again::

    docker compose start

To remove all containers (volumes are kept)::

    docker compose down

To view the logs::

    docker compose logs -f

You might also occasionally need to run other commands inside the containers,
e.g.:

.. code-block:: bash

    docker compose exec web python3 manage.py migrate
    docker compose exec --user root web /bin/bash
    docker compose exec db psql wger -U wger
    docker compose exec cache redis-cli FLUSHALL

Auto-start with systemd
-----------------------

Once your installation is running, you'll typically want it to auto-restart
when the server reboots. With systemd, create
``/etc/systemd/system/wger.service`` (verify the absolute path of the docker
binary with ``which docker``):

.. code-block:: ini

    [Unit]
    Description=wger docker compose service
    PartOf=docker.service
    After=docker.service

    [Service]
    Type=oneshot
    RemainAfterExit=true
    WorkingDirectory=/path/to/the/docker/compose/
    ExecStart=/usr/bin/docker compose up -d --remove-orphans
    ExecStop=/usr/bin/docker compose down

    [Install]
    WantedBy=multi-user.target

Reload systemd and start the service::

    systemctl daemon-reload
    systemctl start wger

If ``systemctl status wger`` shows the service as active (this can take a
moment), everything's good. Enable auto-start on reboot with::

    systemctl enable wger
