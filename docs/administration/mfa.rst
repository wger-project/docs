.. _mfa:

Two-factor authentication
=========================

wger supports two-factor authentication (2FA) and passkeys via
``django-allauth``'s MFA module. Users can manage their factors under
:menuselection:`Preferences --> Two-factor authentication`.

The following factors are supported:

- **TOTP:** time-based one-time passwords from authenticator apps such
  as Aegis, Google Authenticator, 1Password, Bitwarden, etc.
- **Recovery codes:** single-use backup codes generated when TOTP is
  activated.
- **WebAuthn / Passkeys:** hardware security keys (YubiKey, etc.) and
  platform authenticators (Touch ID, Windows Hello, Android biometrics).
  Passkeys can be used both as a second factor and for passwordless
  login (a "Sign in with a passkey" button on the login page).


HTTPS requirement for passkeys
------------------------------

Note that WebAuthn / passkeys **require HTTPS**. Browsers refuse to register or
use credentials on plain HTTP origins (with the exception of ``localhost``
during development). If your instance is served over HTTP, the passkey UI
will be visible but registration and login attempts will fail in the browser.

If you cannot terminate TLS in front of wger, disable the WebAuthn factor as
described below.


Disabling factors
-----------------

Self-hosted instances can restrict the available factors via the ``MFA_SUPPORTED_TYPES``
environment variable. It accepts a comma-separated list and defaults to all
three factors::

    MFA_SUPPORTED_TYPES=totp,recovery_codes,webauthn

To run without WebAuthn (e.g., on an HTTP-only instance)::

    MFA_SUPPORTED_TYPES=totp,recovery_codes

To disable 2FA entirely, leave the variable empty::

    MFA_SUPPORTED_TYPES=

Removing a factor only affects new registrations; existing factors on user accounts
remain in the database. If you re-enable the factor later, users can use them
again without re-enrolling.
