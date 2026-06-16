.. _development:

Development
===========

For developers who want to work on wger itself. The application is split across
three main repositories (backend, frontend, mobile app), plus a few supporting
topics for the development workflow.

.. toctree::
   :maxdepth: 1
   :hidden:

   architecture
   docker
   backend
   frontend
   mobile_app
   celery
   i18n
   dummy_data
   release

:doc:`architecture`
    Overview of the different components and services

:doc:`docker`
    **Recommended starting point.** A Docker-based dev environment for the
    Django backend, no local Python install needed.

:doc:`backend`
    Native install of the Django backend on your machine. Useful if you prefer
    to debug directly from your IDE without going through a container.

:doc:`frontend`
    Set up a development environment for the web frontend.

:doc:`mobile_app`
    Set up a development environment for the mobile app.

:doc:`celery`
    Set up Celery for running background tasks (development and bare-metal
    installations).

:doc:`i18n`
    Manage and update translation files.

:doc:`dummy_data`
    Generate dummy data for development and testing.

:doc:`release`
    Steps for cutting a new backend release.
