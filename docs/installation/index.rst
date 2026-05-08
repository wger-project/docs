.. _production:

Installation
============

It is *very* recommended to use the Docker image for production: everything is
already set up and configured, and it's what we use for the official wger.de
server. The other installation paths exist for cases where Docker isn't an
option, but they're not as extensively tested or are submitted by a third party.

.. toctree::
   :maxdepth: 1
   :hidden:

   docker
   from-source
   truenas
   kubernetes
   proxmox

:doc:`docker`
    Recommended production setup, using Docker and Docker Compose.

:doc:`from-source`
    Installation from source, when Docker isn't an option.

:doc:`truenas`
    Community-contributed installation guide for TrueNAS SCALE.

:doc:`kubernetes`
    Helm charts for deploying to a Kubernetes cluster.

:doc:`proxmox`
    Pointer to a community-maintained Proxmox installation script.
