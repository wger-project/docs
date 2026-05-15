.. _production:

Installation
============

It is *very* recommended to use the Docker image for production: everything is
already set up and configured, and it's what we use for the official wger.de
server. The other installation paths exist for cases where Docker isn't an
option, but they're not as extensively tested or are submitted by the community.

.. toctree::
   :maxdepth: 1
   :hidden:

   docker
   kubernetes
   from-source
   truenas
   proxmox

:doc:`docker`
    Recommended production setup, uses Docker and Docker Compose.

:doc:`kubernetes`
    Helm charts for deploying to a Kubernetes cluster.

:doc:`from-source`
    Installation from source, when Docker isn't an option.

**Community-contributed installation guides:**

:doc:`truenas`
    Installation guide for TrueNAS SCALE.

:doc:`proxmox`
    Proxmox installation script.
