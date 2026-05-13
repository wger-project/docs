.. _mobile_app:

Mobile App
==========

There is a mobile app available for Android and iOS. It's a flutter application
and the source code can be found at

https://github.com/wger-project/flutter


1) Install flutter (the app currently uses the flutter version pinned in
   `action.yml <https://github.com/wger-project/flutter/blob/master/.github/actions/flutter-common/action.yml>`_)

https://docs.flutter.dev/get-started/install

2) Start a wger server. You can either run your own instance (see e.g.
   :ref:`development_docker`) or use the test server:

* URL: ``https://dev.wger.de``
* username: ``user``
* password: ``flutteruser``
* API key: ``31e2ea0322c07b9df583a9b6d1e794f7139e78d4``

The db is reset every day, so feel free to edit or delete anything.


3) Start the application with ``flutter run`` or use your IDE

Please note that depending on how you run your emulator you will need to change
the IP address of the server (on iOS ``http://localhost:8000`` while on android
you need ``http://10.0.2.2:8000``).


Running the tests
-----------------

The unit and widget tests run with::

  flutter test

