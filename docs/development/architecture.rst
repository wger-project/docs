.. _architecture:

Architecture overview
=====================

wger is split across four user-facing components, plus a few supporting
services. This page gives only a broad overview, for details see
:doc:`backend`, :doc:`frontend`, :doc:`mobile_app`


::

    +-----------+       REST       +-----------+
    |   Web     | <--------------> |           |
    | frontend  |                  |           |          +-----------+
    +-----------+                  |   Django  | <------> | Postgres  |
                                   |  backend  |          +-----------+
    +-----------+       REST       |           |              ^
    |  Mobile   | <--------------> |           |              | logical
    |   app     |                  +-----------+              | replication
    |           |                       ^                     v
    |           |                       |              +-------------+
    |           |                       +--------------| PowerSync   |
    |           | <---------- sync stream -----------> |  service    |
    +-----------+                                      +-------------+

Components
----------

**Django backend** (``wger-project/server``)
    The source of truth. REST API under ``/api/v2/``, web UI for browser users,
    admin tooling, exercise and ingredient databases. Talks to Postgres for
    application data and Redis for caching/Celery brokering. See :doc:`backend`.

**Web frontend** (``wger-project/react``)
    The web UI for managing routines, nutrition plans and so on. Pure REST
    client against the Django backend. See :doc:`frontend`.

**Mobile app** (``wger-project/flutter``)
    Flutter app with offline-first storage. Saves data in a local sqlite db
    which the powersync service keeps in sync with the backend's database. For
    a handful of things that need some kind of backend processing, the app
    falls back to the REST API. See :doc:`mobile_app`.

**Postgres**
    Application database.

**PowerSync service**
    The sync engine. Reads from Postgres via a replication slot, and serves a
    sync protocol to the mobile app. Note that this service needs to be reachable
    directly by the client (this is configured in the docker setup via a reverse
    proxy).

**Redis** *(optional)*
    Cache backend and Celery broker.

**Celery worker and beat scheduler** *(optional)*
    Runs background or longer running jobs such as syncing exercises and
    ingredients from upstream, calculates trophy updated, etc.


REST API and auth
-----------------

All client/server traffic (web, mobile, plus the upload path of PowerSync)
goes through the API under ``/api/v2/``. Authentication has three coexisting
surfaces:

**Session cookie**
    Used by the web UI and handles login, signup, MFA, social login,
    password reset and email confirmation. Same backend, different views.

**Permanent token** (``Authorization: Token <key>``)
    Permanent token, created by the user on the web "API key" page. Intended
    for scripts and integrations. There is no refresh, the token is valid until
    manually deleted or a new one is generated.

**JWT** (``Authorization: Bearer <jwt>``)
    Short-lived access token + rotating refresh token. Can be generated manually
    or via ``django-allauth``'s 2FA-aware workflow

See :doc:`/api/api` for details.

Mobile app data flow
--------------------

The powersync service we use gives the mobile app bidirectional sync between
a local db and the backend's one without going through every REST endpoint:

**Download path (server → device).** A dedicated service attaches to Postgres via
a logical replication slot. Each WAL event is evaluated against a set of rules
that defines per-user buckets (e.g. the own weight entries) as well as common
ones (the exercises). The client keeps a stream open, receives updates, and
writes them into the local sqlite file. The Flutter UI can then  watch and
stream these, so the UI updates as soon as new rows arrive or change.

**Upload path (device → server).** Local mutations land in PowerSync's
on-device CRUD queue. A connector drains it and sends each operation to
the django backend. There a dispatcher looks up the handler, validates via
the same DRF serializers the REST API uses, and persists the data.
