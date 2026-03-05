.. _errors_and_pitfalls:

Common error and pitfalls
=========================

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

For more information, consult `django's documentation <https://docs.djangoproject.com/en/4.1/ref/settings/#secure-proxy-ssl-header>`_.

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
  `documentation <https://docs.djangoproject.com/en/4.1/ref/settings/#secure-proxy-ssl-header>`_
  as there are some security considerations.


Wrong pagination links
----------------------

(note that this mostly applies if you are running your own reverse proxy)

For the application to work correctly, you need to make sure that the reverse
proxy is correctly configured.

If this is not the case, some features might break in subtle ways. E.g. the
pagination links in the api are constructed by django from the passed headers.
In this case you might be able to correctly reach and fetch some data but the
"next" link might then point to "localhost", which will not work, only work
while in your home network, etc.

Make sure that you forward the host header as well as the protocol header to
the application. Consult the `Django docs <https://docs.djangoproject.com/en/6.0/ref/request-response/#django.http.HttpRequest.get_host>`_
for details.
