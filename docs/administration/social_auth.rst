.. _social_auth:

Third party logins
==================

The application supports login via social authentication providers using OAuth2.
This allows users to log in using their existing accounts from the appropriate
providers. You can use any provider supported by ``django-allauth``, see the full
list at https://docs.allauth.org/en/latest/socialaccount/providers/index.html

To enable social authentication follow these steps:

**1. Register the application with the provider**

Register your application with the desired social authentication provider to
obtain the necessary credentials (client ID and client secret). You will also
need to configure the exact domain that you will be using. The allauth docs
linked above contain provider-specific setup hints.

**2. Configure the application**

Configure the providers you want to load via the ``WGER_SOCIAL_PROVIDERS``
environment variable, e.g. ``WGER_SOCIAL_PROVIDERS=google,github,gitlab``,
use `the allauth provider IDs <https://github.com/pennersr/django-allauth/tree/main/allauth/socialaccount/providers>`_
for the names.

**3. Add the provider to the database**

Add the provider to the local db with the credentials from the first step, use a
Django shell (``./manage.py shell``) for this. To list, edit or delete entries
later, use the same models, and make sure to keep the enabled providers matching
the entries in the db.

   .. code-block:: python

       from allauth.socialaccount.models import SocialApp
       from django.contrib.sites.models import Site

       app = SocialApp.objects.create(
           provider='google',
           name='Google',
           client_id='...',
           secret='...',
       )
       app.sites.add(Site.objects.get(id=1))

       # To delete
       SocialApp.objects.get(provider='google').delete()

       # To update
       app = SocialApp.objects.get(provider='google').update(client_id='...')



Each provider can be configured via the ``settings`` field, but this is usually
not necessary as they all ship with sensible defaults (scopes, PKCE, etc.).
If you need to configure this, just save a dictionary to the field:

   .. code-block:: python

       app.settings = {
           'scope': ['profile', 'email', 'https://www.googleapis.com/auth/calendar.readonly'],
           'auth_params': {'access_type': 'offline'},
       }
       app.save()

See the allauth provider documentation for the keys each provider supports.
