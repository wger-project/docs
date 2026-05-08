.. _mobile_app:

Mobile App
==========

There is a mobile app available for Android and iOS. It's a flutter application
and the source code can be found at

https://github.com/wger-project/flutter


**Building the app:**

1) Install flutter (the app currently uses the flutter version in .github/actions/flutter-common/action.yml)

https://docs.flutter.dev/get-started/install

2) Start a wger server. You can either run your own instance
(see e.g. :ref:`development_docker`) or use the test server:

* URL: ``https://wger-master.rge.uber.space``
* username: ``user``
* password: ``flutteruser``
* API key: ``31e2ea0322c07b9df583a9b6d1e794f7139e78d4``

The db is reset every day, so feel free to edit or delete anything.


3) Start the application with ``flutter run`` or use your IDE

Please note that depending on how you run your emulator you will need to change
the IP address of the server.

You can run the tests with the ``flutter test``



