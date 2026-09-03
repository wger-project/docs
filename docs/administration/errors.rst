.. _errors_and_pitfalls:

Common errors and pitfalls
==========================

Missing static files
--------------------

If you start the application and don't see any CSS styles, images, etc., there's a
problem with the static files. This happens often.

The reason for this is that for performance reasons, django itself does not serve
any of the static files in production. Instead, it relies on a separate dedicated
process server (in our case, the nginx service, but it could be an external CDN
or similar) to do it. The process works in two steps:

* **Collect:** the django service runs a command to gather all static files into a
  single directory. This step happens automatically when you start the ``web``
  service but can be manually triggered with ``docker compose exec web python3
  manage.py collectstatic``.
* **Serve:** the nginx service reads files from that exact same directory and
  serves them to the user.

These two steps need access to shared docker volume. If you change the volume
configuration, django might not be able to write the file or the web server might
no longer find them. This can happen very easily if you use mounted folders and
the permissions aren't correctly set (make sure they are ``chown``-ed to the UID
and GID 1000, even if this user doesn't exist on your system, and are readable by
everyone).

The solution is always to ensure that the volume or folder which holds your
static files is correctly mounted by both the django and the web server services.
If you want to use your own, exising, web server, you need to make sure that
the files are read and served under the right URLS. Take a look at our nginx.conf
to see how this can look like.

For more information, consult `django's documentation <https://docs.djangoproject.com/en/dev/ref/settings/#secure-proxy-ssl-header>`_.

Email verification links
------------------------

When self-hosting, verification emails may otherwise contain links like
``http://localhost/...``. Set ``SITE_URL`` in your environment to your public
base URL (no trailing slash) so links use your domain instead:

.. code-block:: bash

   SITE_URL=https://your.public.domain

This value is used by the application to build absolute links in outgoing
emails and other places where a full URL is required.

CSRF errors
-----------

You will most probably run into CSRF errors when you try to use the application,
specially if you configured a domain and django's
`CSRF protection <https://docs.djangoproject.com/en/dev/ref/csrf/>`_ kicks in.
To solve this, update the env file and either

* manually set a list of your domain names and/or server IPs
  ``CSRF_TRUSTED_ORIGINS=https://my.domain.example.com,https://118.999.881.119:8008``
  If you are unsure what origin to add here, set set ``DJANGO_DEBUG`` to true,
  restart the service and the error message will tell you exactly which one
  django has a problem with. Note: the port is important!
* or set the ``X-Forwarded-Proto`` header like in the example and set
  ``X_FORWARDED_PROTO_HEADER_SET=True``. If you do this consult the
  `documentation <https://docs.djangoproject.com/en/dev/ref/settings/#secure-proxy-ssl-header>`_
  as there are some security considerations.


Wrong pagination links
----------------------

(note that this mostly applies if you are running your own reverse proxy)

The application builds absolute URLs (for example the "next" links in the API
pagination) from the headers of the incoming request. When these headers get
lost on the way through a reverse proxy, those URLs point to the wrong host or
use ``http`` instead of ``https``. Some features then break in subtle ways:
pagination links might point to "localhost" or only work inside your home
network, and the mobile app shows a warning at login ("Server misconfiguration
detected, headers are not being passed correctly").

Two things need to be in place:

1. The reverse proxy must forward the host and protocol headers to the
   application. The nginx service in the docker compose setup already does
   this, but if you put your own proxy in front of it (or replace it), make
   sure it sets these headers::

      proxy_set_header Host $http_host;
      proxy_set_header X-Forwarded-Proto $scheme;
      proxy_set_header X-Forwarded-Host $http_host;

   When chaining more than one proxy, the outermost one must set the headers
   and the inner ones must pass them through unchanged (see the
   ``X-Forwarded-Proto`` handling in our nginx.conf for an example).

2. The application ignores these headers unless you explicitly allow them.
   Set the following in your env file and recreate the containers::

      X_FORWARDED_PROTO_HEADER_SET=True
      USE_X_FORWARDED_HOST=True

   See :doc:`settings` for details and the security considerations.

To check that everything works, open ``https://your.wger.url/api/v2/exercise/?limit=1``
in your browser: the ``next`` link in the response has to start with exactly
the URL and protocol (http / https) you use to reach the application.


Mobile app stuck on "Connecting"
--------------------------------

If you can log into the mobile app, but the sync status stays on "Connecting"
forever and your data never show up, something on your network is most likely
blocking the connection the app uses to synchronise it. Typical culprits are VPN
apps, ad blockers such as Pi-hole or AdGuard, antivirus apps and some mobile
providers. Try switching between WiFi and mobile data, or turn off the
VPN or ad blocker for a moment and check whether the sync starts working.

A reverse proxy of your own can look exactly the same: if it buffers the
response and the data to sync is small enough to fit into its buffer, nothing
reaches the app at all. The giveaway is the sync service logging
``checkpoint_complete`` for that client while the app is still on "Connecting",
see :ref:`powersync_proxy`.

Mobile app reports "Sync Service Unavailable"
---------------------------------------------

This message means the app can reach your wger server (so logging in works), but
not the PowerSync service that handles the offline synchronisation. The usual
suspects:

* ``SITE_URL`` is wrong: the server tells the app to sync against ``SITE_URL``
  plus ``/ps/``, so a wrong default, a missing port or a domain your phone
  can't reach all break the sync. Set it to the exact URL you use to reach
  the application and recreate the containers. Check the current value by
  opening ``https://your.wger.url/api/v2/powersync-token`` in a browser (logged
  in): the ``powersync_url`` field is the address the app will use.
* The app doesn't go through nginx: the sync service is only reachable through
  the nginx service under ``/ps/``. Don't point the app (or ``SITE_URL``)
  directly at the django container.
* Your own reverse proxy doesn't forward ``/ps/`` or cuts off long-running
  connections, see :ref:`powersync_proxy`.
* The PowerSync container isn't running: check ``docker compose ps`` and
  ``docker compose logs powersync``. You might be using an older docker compose
  setup without the powersync service.

To test that the service is configured correctly, open ``https://your.wger.url/ps/probes/liveness``
with your browser. A short status response means the sync service is reachable.


Mobile app syncs, but never completes
-------------------------------------

A subtler failure: the app logs in and looks connected, but data logged on the
server never appears on the phone or the initial sync stalls just short of 100%.

The usual cause is **response buffering** on a reverse proxy of your own in
front of wger: the sync stream is collected by the proxy instead of being
passed through, so the app never receives the data and keeps retrying. See
:ref:`powersync_proxy` for the settings the ``/ps/`` location needs.
