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

Install node and yarn (1.x), then run::

  yarn install

Then, in the project directory, you can run::

  yarn start

and open http://localhost:3000 in the browser.

To run the tests::

  yarn test


Building for production
-----------------------

To build the app for production, run::

  yarn build

This creates a production bundle and copies it over the the django static folder.
This assumes that the django server folder is next to the react one and is called
"server".


Rendering in django
-------------------

We don't render the whole page with react, just a part of it. We use ``ReactView``
in django to render an emtpy div with a known ID and then let react take over.

Take a look at ``src/index.tsx`` to see how we do this.