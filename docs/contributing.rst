Contributing
============

First off, thanks for taking the time to contribute! ❤️

Our goal is to build an awesome and flexible fitness and nutrition manager,
along with a comprehensive list of exercises and ingredients, all released
under a free license.

All types of contributions are encouraged. And if you like the project but
just don't have time to contribute, that's fine. There are other easy ways
to support the project and show your appreciation, which we would also be
very happy about:

- Talk about it on social media, at local meetups, or tell your friends/gym bros
- Consider :ref:`supporting us with a donation <donation>`
- Star the project on GitHub


Code
----

Obviously, you can also contribute code. Before starting working on a new feature,
please open an issue to discuss it with us. This is important to avoid duplicating
work and to make sure that your contribution is in line with the project's goals.
A good starting point could be the roadmap for the next release:

    https://github.com/orgs/wger-project/projects/5

This application has three main repositories, each with its own purpose (and
quirks) and its own section describing how to set up a development environment:

.. toctree::
   :maxdepth: 1

   development/backend
   development/frontend
   development/mobile_app

In any case you should have a basic grasp of git and GitHub, as well as how to
create pull requests. If you are not familiar with these concepts, please consult
one of the many online resources available, such as

* https://docs.github.com/en/pull-requests
* https://docs.github.com/en/get-started/git-basics

Make sure to always use a feature branch, even if the change is minor.

Once you have the code ready:

* make sure the tests are running (``python3 manage.py test`` in the case of
  python). At the latest you will notice they are failing when you open the pull
  request, but it is better to check them before.
* if you write new code, write new tests. These don't need to test absolutely
  everything, but they should cover the most important parts of the code. If you
  are not sure what to test, just ask us.
* make sure the code is formatted correctly (``ruff format && isort .`` for python)
  and has a line length of 100 characters or less.
* think about UI/UX. If you are adding a new feature, make sure it is easy to use and
  understand. Nobody here is a designer, but we try our best!
* finally open a new PR. You can expect a response from a maintainer within a week, if
  you haven’t heard anything by then, ping the thread.

Also, these are mostly guidelines, not rules. As everywhere in life, use your
best judgment, and feel free to propose changes to this document in a pull request.

Is this the first time you contribute to an open source project? No problem!
Feel free to ping us if you need help setting everything up, it can be very
overwhelming at first. We are happy to help you get started.

Exercises
---------

You can contribute new exercises, images or videos, and edit or translate the
existing ones. These contributions are just as important as code contributions,
as they help improve the overall quality and usability of the application. Please
use the search before to make sure your exercise doesn't already exist.

Note that your account must be at least 3 weeks old and have a verified email.


Translations
------------
You can help translate the application online using Weblate and help make
the application more accessible. To start just visit

    https://hosted.weblate.org/engage/wger

.. _donation:
Support the Project
-------------------
This project is free and open-source, but running it isn’t! Your support helps
keep the server running, funds new development, and improves the overall experience
for everyone. If you enjoy using this app, please consider making a small
contribution. Every bit helps!

  https://www.buymeacoffee.com/wger

Thank you for supporting open-source fitness & nutrition tools! 🙌