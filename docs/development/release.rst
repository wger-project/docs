.. _release_process:

Release process
===============

wger has three components that release independently: the Django backend,
the JS/CSS frontend package, and the Flutter mobile app. The processes are
similar in spirit but have their own steps.

Backend
-------

Before releasing a new (major or minor) version of the backend, a few manual
steps are still necessary. Some of these could and should be automated; for
now they are not, which is also why there aren't that many releases.

Bump versions
~~~~~~~~~~~~~

Bump the app version as well as the minimum required mobile app version in:

* ``wger/version.py``
* ``package.json`` (not strictly required, but keep it in sync)
* All the ``.github/workflows/docker-*.yml`` files
* ``docs/conf.py`` (in the docs repo)

Update the project ID in ``docs/contributing.rst`` (in the docs repo).

Update the contributors list
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Run the script that updates the contributors list::

    python3 extras/authors/generate_authors_api.py

Update the exercise fixture
~~~~~~~~~~~~~~~~~~~~~~~~~~~

It's recommended to update the exercise fixture before a release. Extract it
from a current database, split the files, and copy them as appropriate::

    python ./manage.py dumpdata --indent 4 --natural-foreign exercises > extras/scripts/data.json
    cd extras/scripts/
    python3 filter-fixtures.py
    cp categories.json ../../wger/exercises/fixtures/

Update translations
~~~~~~~~~~~~~~~~~~~

Update the .po files as described in :ref:`i18n`.

Tag the release
~~~~~~~~~~~~~~~

Create a new tag for the release::

    git tag -a 1.2.3 -m "Release 1.2.3"
    git push origin 1.2.3

Create a final tag for the Docker image without the ``-dev`` suffix (after the
images have been built)::

    docker login
    docker buildx imagetools create --tag wger/server:1.2.3 wger/server:latest

Create a GitHub release
~~~~~~~~~~~~~~~~~~~~~~~

Create a new release on GitHub from the tag. Generate the description from the
pull requests and edit as needed, then link to it from the changelog in the
docs repo::

    gh release create "X.Y" --generate-notes

Announce it
~~~~~~~~~~~

Write an announcement and post it on Discord, Mastodon and similar channels.


Frontend
--------

Update the version in ``package.json`` to use the current date::

    NEW_VERSION=$(date +%Y-%m-%d)
    npm version "${NEW_VERSION}" --no-git-tag-version

Publish the new version to npm by manually triggering the ``publish`` workflow
in the GitHub Actions tab.

In the Django server, update the version in ``package.json`` to the same
version and run::

    npm install


Mobile app
----------

The release itself is automated with GitHub Actions and fastlane, but a few
manual preparation steps are involved.

Preflight checks
~~~~~~~~~~~~~~~~

**1) Bump versions**

*Flutter:* If we use a new flutter version, update the version in
``.github/actions/flutter-common/action.yml`` as well as ``flatpak-flutter.yml``
in the ``de.wger.flutter`` repository (commit to master).

*Min server version:* If the new version of the app requires a new minimum
server version, update the version in ``MIN_SERVER_VERSION``.

**2) Verify Apple builds**

Verify that builds succeed with::

    flutter build macos --release
    flutter build ios --release --no-codesign

**3) Dry-run release before uploading**

We use `fastlane <https://fastlane.tools/>`_ to automate the release process.
To test that the update works, you can do a dry-run (needs the various
publishing keys available):

* Increase the build number in ``pubspec.yaml`` (revert after the dry-run was
  successful): ``flutter pub run cider bump build``
* ``flutter build appbundle --release``
* ``bundle install``
* ``bundle update fastlane``
* ``bundle exec fastlane android test_configuration``

It might be necessary to repeat these steps if ``upload_to_play_store``
returns errors such as a missing title.

If a language was added via the Weblate UI, it might be necessary to set the
correct language code:
https://support.google.com/googleplay/android-developer/answer/9844778?hl=en#zippy=%2Cview-list-of-available-languages

**4) Flatpak / Flathub**

The ``de.wger.flutter`` repository is a fork of the flathub metadata repo and
contains the build instructions for the flatpak version of the app.

If there's a new version of sqlite (very likely), make sure the
``flatpak-flutter`` script supports it. A PR upstream might be needed; until
then, the ``de.wger.flutter.yml`` can be generated locally. After making any
changes to ``flatpak-flutter.yml``::

    git clone https://github.com/TheAppgineer/flatpak-flutter.git
    git clone https://github.com/wger-project/de.wger.flutter.git
    cd de.wger.flutter
    ../flatpak-flutter/flatpak-flutter.py --app-module wger flatpak-flutter.yml

    # optional, only needed if you get an error with the flatpak-builder command
    sudo apt install elfutils
    flatpak --user remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

    flatpak-builder --repo=repo --force-clean --sandbox --user --install --install-deps-from=flathub build de.wger.flutter.json
    flatpak run de.wger.flutter

All these steps need to happen on a Linux (virtual) machine.

**5) Update screenshots**

There are different screenshots for the different platforms and languages.
They live in ``fastlane/metadata/android/`` and can be automatically
generated. Consult ``integration_test/README.md`` in the flutter repo.

Takeoff
~~~~~~~

**1) Trigger a release**

Trigger the release manually on GitHub (click "run workflow", use the
``x.y.z`` format for the version). This sets the given version in
``pubspec.yaml``, creates a tag, and bumps the build number. The workflow
then builds the app for the different platforms and uploads it to the Play
Store as well as to a newly created release on GitHub.

https://github.com/wger-project/flutter/actions/workflows/make-release.yml

**2) Merge pull requests**

* In the flathub repo:
  https://github.com/flathub/de.wger.flutter/compare/master...wger-project:de.wger.flutter:master
* In the fork, sync master: https://github.com/wger-project/de.wger.flutter

**3) Update F-Droid**

F-Droid usually auto-updates when it sees a new tag in the repository, but it
might be necessary to manually update the metadata file and open a pull
request:

https://gitlab.com/fdroid/fdroiddata/-/blob/master/metadata/de.wger.flutter.yml
