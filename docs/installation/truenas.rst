.. _truenas:
TrueNAS SCALE
==============

.. note::
    this guide was provided by  `@Phyxsius7 <https://github.com/Phyxsius7>`_ and
    was copied over from `issue 130 <https://github.com/wger-project/docker/issues/130>`_


This guide shows how to install wger on TrueNAS SCALE using your own datasets
(no ixVolume) and no shell. It covers only dataset settings and wger app settings.

Dataset layout
--------------

Create these four child datasets under your preferred parent (example shown):

.. code-block:: none

    /mnt/tank/configs/wger/
    ├─ static
    ├─ media
    ├─ beat
    └─ pgdata

Mount only the four children in the app. The parent wger is for organization.

Step 1 -- Create datasets (UI)
------------------------------

1.  Datasets -> select your parent (e.g., ``tank/configs``).
2.  Add Dataset -> Name: wger -> Save.
3.  Select ``.../wger`` -> Add Dataset four times for: ``static``, ``media``, ``beat``, ``pgdata``. Keep *Share Type = Generic* and defaults.

Step 2 -- Set dataset permissions (UI)
--------------------------------------

Do the following *for each child*: ``static``, ``media``, ``beat``, ``pgdata``.

1.  Datasets -> select the dataset -> ... -> Edit Permissions.
2.  If you see "Set ACL", click it and "Strip ACL" (switch to Unix / POSIX mode).
3.  In "Unix Permissions Editor":
    * Leave "User" and "Group" as they are (don't change owners).
    * Under "Access", check ``Read + Write + Execute`` for ``User / Group / Other`` -> this is 0777.
    * Tick "Apply permissions recursively".
4.  Save.

**Rationale:** wger/NGINX run as a non-root user (UID 1000) and Postgres as UID
999. ``0777`` ensures all containers can write without managing UIDs.

**Optional:** (You can harden later by using ACL entries for UID ``1000`` on ``static/media/beat`` and UID ``999`` on ``pgdata``.)

Step 3 -- Wger app settings (UI)
--------------------------------

Open Apps -> Available Applications -> Wger -> Install and set:

A) Wger Configuration

* **Timezone**: ``Europe/Amsterdam`` (or your TZ)
* **Database Password / Redis Password / Secret Key / Signing Key**: enter values (50 chars for keys)
* **Trusted Origins**: must include at least one full URL you'll use, e.g. ``http://<truenas-ip>:30370`` (Include scheme and port. Add your hostname here too if you'll use one.)

B) Network Configuration

* **WebUI Port** -> Port Bind Mode: Publish
* **Port Number**: choose a **free** port, e.g. ``30370``.

C) Storage Configuration

For each storage below:

* **Type**: Host Path
* **Enable ACL**: OFF
* **Host Path**: point to the matching dataset

.. list-table::
   :header-rows: 1

   * - Storage field
     - Host Path
     - Extra toggle
   * - Wger Static Storage
     - /mnt/tank/configs/wger/static
     - ---
   * - Wger Media Storage
     - /mnt/tank/configs/wger/media
     - ---
   * - Wger Beat Storage
     - /mnt/tank/configs/wger/beat
     - ---
   * - Postgres Data Storage
     - /mnt/tank/configs/wger/pgdata
     - Automatic Permissions = ON ✅

**Automatic Permissions = ON** (only for ``pgdata``) lets SCALE set the correct internal ownership for Postgres on first start.

Click Install

Quick checklist (if install fails)
----------------

* Each child dataset has POSIX 0777 (User/Group/Other all RWX) and was applied recursively.
* Trusted Origins is not empty and includes a full URL with scheme + port.
* The chosen WebUI Port is not used by another app.