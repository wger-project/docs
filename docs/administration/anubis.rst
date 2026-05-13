.. _anubis:

Anubis (anti-AI-crawler proxy)
==============================

For public-facing instances, AI scrapers can and will flood the application with
requests, resulting in high database loads. `Anubis <https://github.com/TecharoHQ/anubis>`_
is a proof-of-work proxy that sits in front of wger and challenges suspicious
clients before they reach the application. The wger.de instance runs
this setup in production.

Legitimate users might see a brief "checking your browser" page on their first
visit, then continue as normal. API clients (including the wger mobile app)
and known search engines are not challenged.

Setup
-----

The docker repo ships everything you need:

* the ``anubis`` service block in
  `docker-compose.override.example.yml <https://github.com/wger-project/docker/blob/master/docker-compose.override.example.yml>`_
* a curated rules file at
  `config/anubis-rules.yml <https://github.com/wger-project/docker/blob/master/config/anubis-rules.yml>`_,
  mounted into the container automatically

Three steps:

1. **Generate an ED25519 private key.** Anubis uses it to sign challenge
   cookies, so don't reuse the example value from the override file.

   .. code-block:: bash

       openssl rand -hex 32

2. **Add a new anubis service to your docker-compose.override.yml.**
   Copy the block from ``docker-compose.override.example.yml``, paste the
   key into ``ED25519_PRIVATE_KEY_HEX`` and set ``WEBMASTER_EMAIL`` to a
   real address.

3. **Point your reverse proxy at Anubis instead of the web service.**

   With Caddy, swap the target in your ``Caddyfile``:

   .. code-block::

       reverse_proxy anubis:3000 {
           header_up Host {host}
           # ... keep your existing header_up directives
       }

   With nginx, change the upstream in ``config/nginx.conf``:

   .. code-block:: nginx

       upstream wger {
           server anubis:3000;
       }

Restart with ``docker compose up -d``. Hitting the site in a browser
should briefly show the Anubis challenge page; ``/api/v2/*`` stays
challenge-free.

Rules
-----

The bundled ``anubis-rules.yml`` is a curated preset that:

* always allows ``/api/v2/*`` (so the mobile app and other API clients
  are never challenged)
* allows well-known good search engine crawlers (Google, Bing,
  DuckDuckGo, and others)
* allows ``robots.txt``, ``favicon.ico`` and ``/.well-known`` routes
* blocks pathological bots and AI scrapers (Anubis's bundled
  ``ai-block-moderate`` preset)
* applies browser-fingerprint heuristics so requests that look like a
  real browser get an easier challenge

Read the full rules at
`config/anubis-rules.yml <https://github.com/wger-project/docker/blob/master/config/anubis-rules.yml>`_.

.. warning::

   The ``allow-api-v2`` rule sits at the top of the file for a reason.
   Anubis evaluates rules top to bottom and stops at the first match. If
   you reorder or remove this rule, the mobile app and other API clients
   will start getting proof-of-work challenges they can't solve, and
   effectively break.

Monitoring
----------

Anubis exposes Prometheus metrics on its ``METRICS_BIND`` port (``:9090``
in the example override). The docker repo ships a Grafana dashboard at
``grafana/dashboards/anubis.json`` that visualises traffic, challenge
rate and pass/block ratio.

For general Grafana setup, see :doc:`monitoring`.

.. note::

   On from-source installations, Anubis runs as a separate service
   alongside gunicorn and your reverse proxy. The wger docs don't cover
   that setup; refer to the `upstream Anubis project
   <https://github.com/TecharoHQ/anubis>`_ for installation instructions.
