.. _auth_proxy:

Authentication Proxy (SSO)
==========================

This feature allows you to delegate user authentication to a trusted reverse
proxy server (e.g., Nginx, Caddy, Apache, Traefik) or an external authentication
provider (e.g., Authelia, Authentik, Keycloak, systems using LDAP, SAML, OAuth2)
fronting the application.

When enabled, if the trusted proxy successfully authenticates a user and sets
a specific HTTP header containing their username, wger will automatically log
that user in. If the user doesn't exist, it can optionally be created automatically.


.. warning::
  Misconfiguration of this feature can lead to severe security vulnerabilities,
  potentially allowing attackers to impersonate any user!


The core risk is that an attacker could directly send the authentication header
(e.g., ``X-Remote-User: admin``) to the application, bypassing the proxy's
authentication, and gain access as that user. Because of this, your reverse proxy
**must** be configured to overwrite the authentication header from any incoming
request before it processes the request, e.g. pseudocode for Caddy::

    your.wger.domain.com {
        # ... authentication configuration (e.g., forward_auth to Authelia/Authentik) ...
        # Assuming auth service sets Upstream-User header upon success

        reverse_proxy http://<wger_backend_ip>:<port> {

            # Remove header
            header_up delete X-Remote-User

            # Set header based on auth service response
            header_up +X-Remote-User {http.reverse_proxy.upstream.header.Upstream-User}
        }
    }


If you plan to use the mobile app, make sure to bypass the ``api/*`` subfolder
from your proxy auth so that the app can access the api endpoint. In caddy,
it might look something like this::

    wger.example.com {
      import cert
      # allow api path to bypass auth (requires api key)
      handle /api/* {
        reverse_proxy wger_server:8000
      }
        # Enclosing in `route` forces execution order
        route {
            # Forward outpost path to actual outpost
            reverse_proxy /outpost.goauthentik.io/* authentik_server:9000
      ...
    }


Settings
--------

``AUTH_PROXY_HEADER``
  *Required.* The name of the HTTP header that the reverse proxy sets with
  the authenticated user's username. Usually ``Remote-User``, but can be any
  name you choose. Default is empty, which disables the feature.

  Apply Django's header-name convention: uppercase, replace hyphens with
  underscores, prefix with ``HTTP_``. So if the proxy sets ``X-Remote-User``,
  configure ``HTTP_X_REMOTE_USER`` here.

``AUTH_PROXY_TRUSTED_IPS``
  *Required.* Comma-separated list of trusted IP addresses from which the
  application accepts authentication headers. Usually this is the IP of your
  reverse proxy. If the proxy runs on the same host, include ``127.0.0.1``
  and/or ``::1``.

``AUTH_PROXY_CREATE_UNKNOWN_USER``
  Default ``False``. Whether to automatically create a new user account
  if the authenticated user does not exist in the database yet.

``AUTH_PROXY_USER_EMAIL_HEADER``
  The header to read the email from for auto-created users. Ignored if
  ``AUTH_PROXY_CREATE_UNKNOWN_USER`` is not set. Same Django header-name
  convention as ``AUTH_PROXY_HEADER``.

``AUTH_PROXY_USER_NAME_HEADER``
  The header to read the display name from for auto-created users. Ignored
  if ``AUTH_PROXY_CREATE_UNKNOWN_USER`` is not set. Same Django header-name
  convention as ``AUTH_PROXY_HEADER``.

