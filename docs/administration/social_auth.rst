.. _social_auth:

Social Auth (OAuth2)
====================

The application supports login via social authentication providers using OAuth2.
This allows users to log in using their existing accounts from the appropriate providers,
such as Google, Facebook, or GitHub.

To enable social authentication follow these steps:

1. Register your application with the desired social authentication provider to
   obtain the necessary credentials (client ID and client secret). You will also need
   to configure the exact domain that you will be using. For google, this is done
   in the cloud console https://console.cloud.google.com/auth/

2. Set ``USE_SOCIAL_AUTH`` to ``True`` in your settings to enable social authentication.

3. Configure the providers with the obtained credentials. For example, for Google:
   ``python manage.py setup_google_oauth --client_id "123456..." --client_secret "7890..."``
