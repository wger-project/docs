.. _oauth2_provider:

OAuth2 provider
===============

wger can act as an OAuth2 provider: other applications let their users log in
with their wger account and then access the API on their behalf. Before that happens,
users are asked on a wger page whether they want to grant the application access.

This is the opposite direction of :doc:`social_auth`, where your users log in
to wger with a Google or GitHub account.

Nothing here is active unless you configure it.

The feature is provided by ``django-allauth``. This page covers a basic setup, for
everything beyond that (all available settings, the client options, self-registering
clients) see `their documentation <https://docs.allauth.org/en/latest/idp/openid-connect/index.html>`_.

Setup
-----

**1. Generate a signing key**

The key signs the identity tokens that are handed to the applications::

    docker compose exec web ./manage.py generate-oidc-key

Paste the output into your env file, and restart the application. Replacing the
key later means the applications have to log their users in again.

**2. Register the application**

Applications are stored in the database, add them from a Django shell
(``./manage.py shell``), the same way as the :ref:`social login providers
<social_auth>`:

.. code-block:: python

    from allauth.idp.oidc.models import Client

    app = Client(name='My application', type=Client.Type.PUBLIC)
    app.set_redirect_uris(['https://example.com/callback'])
    app.set_scopes(['openid', 'profile', 'email', 'api:read', 'api:write'])
    app.set_grant_types([Client.GrantType.AUTHORIZATION_CODE, Client.GrantType.REFRESH_TOKEN])
    app.set_response_types([Client.ResponseType.CODE])
    app.save()

    # This is the client ID the application needs
    print(app.id)

    # To list or delete entries later
    Client.objects.all()
    Client.objects.get(id='...').delete()

The redirect URI has to match what the application expects, exactly. Getting
it wrong is the most common cause of a login that ends on an error page
instead of back in the application.

The scopes are what the application is allowed to ask for, and users see them
as individual checkboxes when they are asked for their consent:

``api:read``
  Read anything the user can see: their workouts, nutrition plans, body
  measurements and progress photos.

``api:write``
  Add, change and delete that same data.

``openid``, ``profile``, ``email``
  Identify the user. These say who is logged in and are what an application
  needs to offer a "log in with wger" button, they open no data.

Only list what the application actually needs. An application that just
displays workouts should not be able to delete them.

Applications that run on the user's device (mobile or desktop apps) are
``PUBLIC`` and identify themselves without a secret. For an application running
on a server, use ``CONFIDENTIAL`` and set a secret. Pick it yourself and note it
down, wger only stores a hash of it and cannot show it to you later:

.. code-block:: python

    app.type = Client.Type.CONFIDENTIAL
    app.set_secret('the secret you hand to the application')
    app.save()

**3. Point the application at your instance**

Most applications only need the base URL of your instance and find the rest on
their own, through the discovery document at
``https://your-instance/.well-known/openid-configuration``. If an application
asks for the individual addresses instead:

===================== ==========================================
Authorization         ``/identity/o/authorize``
Token                 ``/identity/o/api/token``
User info             ``/identity/o/api/userinfo``
Revocation            ``/identity/o/api/revoke``
Public keys           ``/.well-known/jwks.json``
===================== ==========================================

Only the discovery document and the public keys are readable from other
websites. Applications that run entirely in the browser cannot exchange the
login for a token from JavaScript, they need a small backend of their own.

Settings
--------

``IDP_OIDC_PRIVATE_KEY``
  The key from step 1, in PEM format. It doubles as the on/off switch: as long
  as it is empty, applications cannot even start the login flow, and access
  tokens that were handed out earlier stop working. Everything else in wger is
  unaffected either way, in particular the regular API access of the mobile
  app.

Housekeeping
------------

Expired tokens stay in the database until they are cleaned up. This happens
once a day if you have Celery running (see ``USE_CELERY`` in :ref:`settings`),
otherwise trigger it yourself now and then::

    docker compose exec web ./manage.py oidc_cleartokens
