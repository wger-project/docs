.. _release_process:

Release process
===============

wger has four components that release independently: the Django backend,
the JS/CSS frontend package, the Flutter mobile app, and the Python API
client. The processes are similar in spirit but have their own steps.

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

Check the sync rules
~~~~~~~~~~~~~~~~~~~~

If the release changes a table the mobile app syncs, the sync rules in the
docker repository (``services/config-powersync/sync_rules.yaml``) need the
matching change. Mention it in the release notes: the file is bind mounted, so
self-hosted instances only get it by pulling that repository.

A new column needs one thing more. Adding it writes no WAL, so PowerSync never
re-replicates the rows already in its bucket storage and clients read the
column as missing, indefinitely on an instance that never deploys new rules.
Touch every row of the table in the same migration::

    migrations.RunSQL(
        sql=['UPDATE changed_table SET id = id;'],
        reverse_sql=migrations.RunSQL.noop,
    )

Deploying changed rules makes PowerSync reprocess everything and all clients
re-download their buckets, so it is not a substitute for the row touch.

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

    flatpak-builder --repo=repo --force-clean --sandbox --user --install --install-deps-from=flathub build de.wger.flutter.yml
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


Python API client
-----------------

The client in the ``api-client`` repository is generated from the backend's
OpenAPI schema, after each release.

For a released backend, the first two steps are a single click: trigger the
``Regenerate client`` workflow in the Actions tab and it refreshes the schema,
regenerates the client, runs the checks and opens a pull request with the
result. Continue at :ref:`review the contract diff <client_contract_diff>`.
Do the steps by hand when the schema is only available from a local instance,
since the runner cannot reach it.

Refresh the schema
~~~~~~~~~~~~~~~~~~

Point the sync script at an instance running the release you are targeting.
Note that the instance must use PostgreSQL, as Django derives the bounds of
its integer fields from the database backend and SQLite produces a schema
that no real deployment matches::

    uv run python scripts/sync_schema.py --base-url https://wger.de

Running ``./manage.py spectacular --fail-on-warn`` in the backend will tell you
if there are any problems with the schema.

Regenerate the client
~~~~~~~~~~~~~~~~~~~~~

Regenerate and run the tests (api cient repo)::

    ./scripts/generate.sh && uv run pytest

The schema and the generated code belong in the same commit, CI rejects a
schema that moved without a regenerated client.

.. _client_contract_diff:

Review the contract diff
~~~~~~~~~~~~~~~~~~~~~~~~

Every release is tagged and the schema is committed, so the previous release is
the diff base::

    git diff <previous tag> -- schema/wger-openapi.yaml

Look for removed endpoints, changed response types, new required parameters,
and renamed ``operationId`` values. The last ones rename functions in the
generated client, which breaks its consumers even though the backend did not
change.

Bump the version and update the changelog
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Set the new version, which updates ``pyproject.toml`` and ``uv.lock`` together,
since the lock records the project version as well::

    uv version 2.7.0

Note in the readme whether the client still works against the previous backend
release, along with anything that behaves differently there.

Move the ``Unreleased`` entries of ``CHANGELOG.md`` under the new version.
Above all the renames found in the diff, and links to the backend release notes
for the API changes themselves.


Tag and release
~~~~~~~~~~~~~~~

Create and push the tag, then publish a release from it (the workflow triggers
on a published release, not on the tag alone)::

    git tag 2.7.0
    git push origin 2.7.0
    gh release create 2.7.0 --generate-notes