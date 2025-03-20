.. _api:

Using the API
==============

Public endpoints, such as the list of exercises or the ingredients can be
accessed without authentication. For user owned objects such as
routines, you need to authenticate.


For some info on how to create routines over the API, see

.. toctree::
   :maxdepth: 1

   routines


JWT Tokens
----------

This is the suggested way. You generate a temporary token which you send in
the header with each request that needs authorization

1. Get the tokens

Send your username and password to the ``/api/v2/token``
endpoint, you will get an ``access`` and a ``refresh`` token
back::

    result = requests.post(
        'https://wger.de/api/v2/token',
        data={'username': 'user', 'password': 'admin'}
    )
    access_token = result.json()['access']
    refresh_token = result.json()['refresh']

    print(result.json())
    >>> {'refresh': 'eyJhbGciOiJIUzI1...', 'access': 'eyJhbGciOiJIUzI...'}



2. Authenticate

Pass the access token in the Authorization header as ``"Bearer: your-token"``::

    result = requests.get(
        'https://wger.de/api/v2/workout/',
        headers={'Authorization': f'Bearer {access_token}'}
    )

    print(result.json())
    >>> {'count': 5, 'next': None, 'previous': None, 'results': [{'id':.....

Additionally, you can send the access token to ``/token/verify/``
endpoint to verify it::

    result = requests.post('https://wger.de/api/v2/token/verify', data={'token': access_token})

3. Refresh

When this short-lived access token expires, you can use the longer-lived
``refresh`` token to obtain another access token::

    result = requests.post(
        'https://wger.de/api/v2/token/refresh/',
        data={'refresh': refresh_token}
    )
    token = result.json()

    print(token)
    >>> {'access': 'eyJhbGciOiJI...'}


Permanent Token
---------------
You can also pass a permanent token in the header to authenticate, but this
method should be considered deprecated::

    token = 'abcdef123...'
    result = requests.get(
        'https://wger.de/api/v2/routine/',
        headers={'Authorization': f'Token {token}'}
    )

    print(result.json())
    >>> {'count': 5, 'next': None, 'previous': None, 'results': [{'id':.....

To generate a key, visit the API page on the web app.

You can also get a permanent token from the ``login`` endpoint.
Send a username and password in a POST request. If the user doesn't
currently have a token, a new one will be generated for you

Pagination
----------
By default all results are paginated by 20 elements per page. If you want to
change this value, add a ``?limit=<xxx>`` to your query.
You will find in the answer JSON the ``next`` and ``previous``
keywords with links to the next or previous result pages.


Format negotiation
------------------

At the moment only JSON and the browsable HTML view are supported. That
means that you can use the same endpoints from your browser or a client.
Because of this, for the majority of REST clients it will not be
necessary to explicitly set the format, but you have the following options:

* Set the **Accept** header:
    * ``application/json``
    * ``application/json; indent=4`` - useful for debugging, will indent the result
    * ``text/html`` - browsable HTML view, like in the browser

* Set the format **directly in the URL**:
    * ``/api/v2/<endpoint>.json/``
    * ``/api/v2/<endpoint>/?format=json``
    * ``/api/v2/<endpoint>.api/`` - browsable HTML view


Ordering
--------
Simply use ``?ordering=<fieldname>`` to order by that field.
You can also specify more than one field name, just give it a list separated
by commas ``?ordering=<field1>,<field2>``. To reverse
the order use like in django a ``-`` in front of the field.



Filtering resources
-------------------
You can easily filter all resources by specifying the filter queries in the
URL: ``?<fieldname>=<value>``, combinations are possible,
the filters will be AND-joined:
``<f1>=<v1>&<f2>=<v2>``.
Please note that for boolean values you must pass 'False' or 'True' other
values, e.g. 1, 0, false, etc. will be ignored. Like with not filtered queries,
your objects will be available under the 'results' key.

Note that it is not currently possible to specify more than one value, e.g.
category 1 or 2.


