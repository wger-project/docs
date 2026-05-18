Using the API
=============

The wger REST API is served under ``/api/v2/``. It returns JSON by default,
supports filtering, ordering and pagination, and uses standard HTTP status
codes and verbs.

Public endpoints, such as the list of exercises or the ingredients, can be
accessed without authentication. For user-owned objects such as routines,
you need to authenticate.


For specific info on how to create routines over the API, see
:doc:`routines`.


Interactive reference
---------------------

wger ships a complete OpenAPI specification for every endpoint, generated
automatically from the code. The following (interactive) viewers are exposed:

* ``/api/v2/`` returns a JSON listing of every endpoint (great for
  discovery), or renders an HTML index when opened in a browser
* ``/api/v2/schema`` the raw OpenAPI schema in JSON
* ``/api/v2/schema/ui/`` Swagger UI, lets you try requests directly in
  the browser
* ``/api/v2/schema/redoc/`` ReDoc, cleaner read-only view



JWT Tokens
----------

The recommended authentication mechanism. You exchange a username/password
pair for a short-lived access token (10 minutes in the Docker default) and
a long-lived refresh token (120 days), then send the access token in the
header with each authenticated request. The lifetimes are configurable via
the ``ACCESS_TOKEN_LIFETIME`` and ``REFRESH_TOKEN_LIFETIME`` env vars (see
:doc:`/administration/settings`).

1. Get the tokens

   wger does not have a credentials-to-JWT endpoint, since that would bypass
   2FA. The two supported ways to obtain a refresh token are:

   **a) From the allauth-headless login endpoint** (recommended for apps
   that have a proper login UI, and for scripts targeting accounts
   *without* 2FA): ``POST /_allauth/app/v1/auth/login`` with
   ``{"username", "password"}``::

       result = requests.post(
           'https://wger.de/_allauth/app/v1/auth/login',
           json={'username': 'user', 'password': 'admin'},
       )
       data = result.json()['data']
       access_token = data['access']
       refresh_token = data['refresh']

   For accounts *with* 2FA enabled the response is a partial-login: it
   contains an ``X-Session-Token`` header plus a ``requires_mfa`` flag
   instead of the tokens. You then send the TOTP/recovery code to
   ``POST /_allauth/app/v1/auth/2fa/authenticate``, passing the session
   token in the ``X-Session-Token`` header, to receive the real access
   and refresh tokens.

   The full request/response shapes (including the partial-login,
   email-verification and password-reset flows) are documented upstream:

   * Conceptual introduction:
     https://docs.allauth.org/en/latest/headless/introduction.html
   * OpenAPI specification of every endpoint:
     https://docs.allauth.org/en/latest/headless/openapi-specification/

   **b) From the web "API key" page** (recommended for personal scripts
   and long-running integrations, especially when 2FA is enabled): log
   into the web app, open *User settings → API key*, and mint a
   long-lived refresh token. You only see the value once, so store it
   immediately. Because reaching that page requires a regular log-in,
   any 2FA configured on the account is enforced before the token is
   issued.

2. Authenticate

Pass the access token in the Authorization header as ``"Bearer your-token"``::

    result = requests.get(
        'https://wger.de/api/v2/routine/',
        headers={'Authorization': f'Bearer {access_token}'}
    )

    print(result.json())
    >>> {'count': 5, 'next': None, 'previous': None, 'results': [{'id':.....

Additionally, you can send the access token to the ``/api/v2/token/verify``
endpoint to verify it::

    result = requests.post('https://wger.de/api/v2/token/verify', data={'token': access_token})

3. Refresh

When the short-lived access token expires, use the longer-lived ``refresh``
token to obtain a new pair. Both tokens get rotated: the response contains a
fresh access *and* refresh token, and the previous refresh token is
blacklisted immediately. Store the new refresh token for the next cycle::

    result = requests.post(
        'https://wger.de/api/v2/token/refresh',
        data={'refresh': refresh_token}
    )

    print(result.json())
    >>> {'access': 'eyJhbGciOiJI...', 'refresh': 'eyJhbGciOiJI...'}


Permanent Token
---------------
You can also pass a permanent token in the header to authenticate, but this
method is intended for personal scripts or one-off integrations::

    token = 'abcdef123...'
    result = requests.get(
        'https://wger.de/api/v2/routine/',
        headers={'Authorization': f'Token {token}'}
    )

    print(result.json())
    >>> {'count': 5, 'next': None, 'previous': None, 'results': [{'id':.....

To generate a key, log in to the web app and open the "API key" section in
your user settings. The same page also lets you mint a long-lived JWT
refresh token for use with the headless auth surface.

Pagination
----------

By default lists are paginated at 20 elements per page. Override with
``?limit=<n>``. There is no hard cap on the limit, but very large pages
will be slow and may put load on the server. The response JSON contains
``next`` and ``previous`` URLs for navigation, and ``count`` for the
total number of results.


Rate limiting
-------------

A few sensitive endpoints are rate-limited (per IP for anonymous callers,
per user for authenticated ones):

* ``/_allauth/app/v1/auth/login`` and ``/_allauth/app/v1/auth/2fa/authenticate`` (authentication): 10 requests/min
* ``/api/v2/userprofile/`` (registration): 5 requests/min
* ``/api/v2/ingredient/`` and ``/api/v2/ingredientinfo/`` (list): 120 requests/min
* ``/api/v2/ingredient/<id>/`` and ``/api/v2/ingredientinfo/<id>/`` (detail): 300 requests/min
* ``/api/v2/ingredient-sync/`` (bulk sync): 600 requests/min

Exceeding a limit returns HTTP 429 with a ``Retry-After`` header. All other
endpoints are unthrottled.


Format negotiation
------------------

The API returns JSON by default. For clients you usually don't need to set
anything explicitly. To request a specific format:

* Set the ``Accept`` header to ``application/json``, or
  ``application/json; indent=4`` for indented output (useful when debugging).
* Or use a URL suffix: ``/api/v2/<endpoint>.json`` for raw JSON, or
  ``/api/v2/<endpoint>.api`` for the browsable HTML view (same as opening
  the endpoint in a browser).


Ordering
--------

Use ``?ordering=<fieldname>`` to order results by that field. Combine
multiple fields with commas: ``?ordering=<field1>,<field2>``. Prefix a
field with ``-`` to reverse, as in Django: ``?ordering=-date``.



Filtering resources
-------------------

Filter list endpoints by appending query parameters: ``?<fieldname>=<value>``.
Multiple filters are AND-joined: ``?<f1>=<v1>&<f2>=<v2>``.

For boolean fields, pass ``True`` or ``False`` (case-sensitive); other
values like ``1``, ``0`` or ``false`` are ignored.

Specifying multiple values for the same field (e.g., category 1 *or* 2) is
not currently supported.


