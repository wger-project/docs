.. _frontend:

Frontend
========

Note that "frontend" is not completely correct, more like "part of the frontend".
The application uses django to render the main page, react is only used for some
parts of the application that need to be more interactive. These parts have their
own repository which can be found at

https://github.com/wger-project/react

Getting Started
---------------

Copy ``.env.TEMPLATE`` to ``.env.development`` and edit it to your needs.

You can obviously use your own instance, but feel free to use the test server
(the db is reset every day):

* URL: ``https://wger-master.rge.uber.space``
* username: ``user``
* password: ``flutteruser``
* API key: ``31e2ea0322c07b9df583a9b6d1e794f7139e78d4``

Install node (>22) and run::

  npm install

Then, in the project directory, you can run::

  npm start

and open http://localhost:3000 in the browser.

To run the tests::

  npm run test


Release process
---------------

Update the version in ``package.json`` to use the current date::

    NEW_VERSION=$(date +%Y-%m-%d)
    npm version "${NEW_VERSION}" --no-git-tag-version

Publish the new version to npm by manually triggering the workflow ``publish``
in the github actions tab.

In the django server, update the version in ``package.json`` to the same
version and run::

  npm install



Rendering in django
-------------------

We don't render the whole page with react, just a part of it. We use ``ReactView``
in django to render an emtpy div with a known ID and then let react take over.

Take a look at ``src/index.tsx`` to see how we do this.

If you want to test the new version locally, you can use ``npm link`` which will
create a symlink to the dev repository and allow you to instantly see the changes::

  cd /path/to/react/repo
  npm link
  cd /path/to/wger/server
  npm link @wger-project/react-components


Don't forget to unlink everything once you're done::

  cd /path/to/wger/server
  npm unlink @wger-project/react-components
  cd /path/to/react/repo
  npm unlink
