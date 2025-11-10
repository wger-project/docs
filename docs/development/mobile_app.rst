.. _mobile_app:

Mobile App
==========

There is a mobile app available for Android and iOS. It's a flutter application
and the source code can be found at

https://github.com/wger-project/flutter


Building the app
****************

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


Release process
***************

The release process itself is automated with github actions and fastlane, however
there are some manual steps involved to prepare the release:


Preflight checks
----------------

**1) Bump flutter version**

If we use a new flutter version, update the version in ``.github/actions/flutter-common/action.yml``
as well as ``flatpak-flutter.json`` in the ``de.wger.flutter`` repository (commit to master).

**2) Update cocoapods for iOS and Mac.**

In the ``ios`` and ``macos`` folders run ``pod update`` (or ``gem update``). Check that builds
are possible with ``flutter build macos --release`` and ``flutter build ios --release --no-codesign``.

**3) Dry-run release before uploading**

We use `fastlane <https://fastlane.tools/>`_ to automate the release process. To
test that the update works, you can make a dry-run (needs the different
publishing keys available)

* Increase build nr in pubspec.yaml (revert after the dry-run was successful):
  ``flutter pub run cider bump build``
* ``flutter build appbundle --release``
* ``bundle install``
* ``bundle update fastlane``
* ``bundle exec fastlane android test_configuration``

It might be necessary to repeat these steps if upload_to_play_store returns any errors
such as a missing title or similar.

Also note that if a language was added over the weblate UI, it might be necessary
to set the correct language code:

https://support.google.com/googleplay/android-developer/answer/9844778?hl=en#zippy=%2Cview-list-of-available-languages


**4) Flatpak / Flathub**

The ``de.wger.flutter`` repository is a fork of the flathub metadata repo and
contains the build instructions for the flatpak version of the app.

If there is a new version of sqlite (very probably), make sure the flatpak-flutter
script supports it. A PR to upstream might be needed in this case, but till then
the de.wger.flutter.json can also be generated locally. After making any changes
to flatpak-flutter.json::

    git clone https://github.com/TheAppgineer/flatpak-flutter.git
    git clone https://github.com/wger-project/de.wger.flutter.git
    cd de.wger.flutter
    ../flatpak-flutter/flatpak-flutter.py --app-module wger flatpak-flutter.json

    # optional, only needed if you get an error with the flatpak-builder command
    sudo apt install elfutils
    flatpak --user remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

    flatpak-builder --repo=repo --force-clean --sandbox --user --install --install-deps-from=flathub build de.wger.flutter.json
    flatpak run de.wger.flutter

Note: all these steps need to happen on a linux (virtual) machine.

**5) Updating screenshots**

There are different screenshots for the different platforms and languages. They
are in fastlane/metadata/android/ and can be automatically generated. To do so,
consult ``integration_test/README.md`` in the flutter repo.

Takeoff
-------

**1) Trigger a release**

The release process must be manually triggered on Github (click on "run workflow", use
use x.y.z format for the version). This will set the given version in pubspec.yaml,
create a tag, and bump the build number. The workflow will then build the app for
the different platforms and upload the file to the Play Store as well as to a newly
created release on Github.

https://github.com/wger-project/flutter/actions/workflows/make-release.yml

**2) Merge pull requests**

* in the flathub repo:
  https://github.com/flathub/de.wger.flutter/compare/master...wger-project:de.wger.flutter:master
* in the fork, sync master https://github.com/wger-project/de.wger.flutter

**3) Update f-droid**

Fdroid usually autoupdates when it registers a new tag in the repository. However,
it might be necessary to manually update the metadata file for the app and open a
pull request:

https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/de.wger.flutter.yml

