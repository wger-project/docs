.. _mobile_app:

Mobile App
==========

There is a mobile app available for Android and iOS. It's a flutter application
and the source code can be found at

https://github.com/wger-project/flutter


Building the app
----------------
1) Install flutter (the app currently uses flutter 3.27)

https://docs.flutter.dev/get-started/install


2) Start a wger server. You can either run your own instance
(see :ref:`development_docker`) or use the test server:

* URL: ``https://wger-master.rge.uber.space``
* username: ``user``
* password: ``flutteruser``
* API key: ``31e2ea0322c07b9df583a9b6d1e794f7139e78d4``

(the db is reset every day, so feel free to edit or delete anything)


3) Start the application with ``flutter run`` or use your IDE
(please note that depending on how you run your emulator you will need to change the IP address of
the server)

You can run the tests with the ``flutter test``


Release process
---------------
The release process itself is automated with github actions and fastlane, however
there are some manual steps involved to prepare the release:


1) Update flutter version

If we use a new version, update the version in

* this file
* Github Actions in ``build-release.yml`` in ``.github/workflows``
* If needed, update the F-droid build recipe in https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/de.wger.flutter.yml

2) Dry-run release before uploading

We use `fastlane <https://fastlane.tools/>`_ to automate the release process. To test the release process,
install it first and then

* Increase build nr in pubspec.yaml (revert after the dry-run was successful)
* ``flutter build appbundle --release``
* ``bundle install``
* ``bundle update fastlane``
* ``bundle exec fastlane android test_configuration`` (needs the different publishing keys available)

It might be necessary to repeat these steps if upload_to_play_store returns any errors
such as a missing title or similar.

Also note that if a language was added over the weblate UI, it might be necessary
to set the correct language code:

https://support.google.com/googleplay/android-developer/answer/9844778?hl=en#zippy=%2Cview-list-of-available-languages

3) Push tags to trigger release

Make sure that the commit that will be tagged was already pushed or didn't change
any dart code, otherwise the automatic linter might push a "correction" commit
and the build step will fail.

Set the vX.Y.Z tag locally, push it and delete it. It will get recreated to X.Y.Z.
by github actions::

  TAG=vX.Y.Z && git tag $TAG && git push origin $TAG && git tag -d $TAG


4) Edit release

If necessary, edit the created release on github, e.g. to generate the release notes from
the pull requests.

5) Merge pull requests

* in the flathub
  repo: https://github.com/flathub/de.wger.flutter/compare/master...wger-project:de.wger.flutter:master
* in the fork sync master https://github.com/wger-project/de.wger.flutter
* in the flutter repo: https://github.com/wger-project/flutter/branches



Updating screenshots
--------------------
There are different screenshots for the different platforms and languages. They
are in fastlane/metadata/android/ and can be automatically generated. To do so,
consult ``integration_test/README.md`` in the flutter repo.
