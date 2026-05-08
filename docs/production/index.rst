.. _production:

Installation
============

It is *very* recommended to use the Docker image for production: everything is
already set up and configured, and it's what we use for the official wger.de
server. The other installation paths exist for cases where Docker isn't an
option, but they're not as extensively tested or supported.

.. toctree::
   :maxdepth: 1
   :hidden:

   docker
   installation
   true_nas

:doc:`docker`
    Recommended production setup, using Docker and Docker Compose.

:doc:`installation`
    Installation from source, when Docker isn't an option.

:doc:`true_nas`
    Community-contributed installation guide for TrueNAS SCALE.
