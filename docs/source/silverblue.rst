Fedora Silverblue
=================

Silverblue is an immutable, image-based Fedora desktop built on **rpm-ostree** -
the OS is a read-only OSTree deployment you update atomically and can roll back.
These are notes from real use, not official docs.

Status and updates
------------------

.. code-block:: console

   $ rpm-ostree status        # current + previous deployments (* = booted)
   $ rpm-ostree upgrade       # stage the latest update; reboot to apply
   $ systemctl reboot

Layering packages
-----------------

Install RPMs on top of the base image (a reboot applies them):

.. code-block:: console

   $ rpm-ostree install vim-enhanced
   $ rpm-ostree uninstall vim-enhanced
   $ rpm-ostree install --apply-live htop   # apply without reboot (experimental)

Keep the base image lean and do most dev work in a container (see ``toolbox``
below) rather than layering everything onto the host.

Rolling back and pinning
------------------------

.. code-block:: console

   $ rpm-ostree rollback      # boot the previous deployment
   $ ostree admin pin 0       # pin the current deployment so it is not GC'd

Rebasing to a new release
------------------------

.. code-block:: console

   $ rpm-ostree rebase fedora:fedora/42/x86_64/silverblue

You can also override a single package from a Bodhi update, e.g. to test a fix:

.. code-block:: console

   $ rpm-ostree override replace --experimental --from repo=updates-testing <pkg>

toolbox - a mutable dev container
--------------------------------

``toolbox`` gives you a Fedora container that shares your home directory, so you
can ``dnf install`` freely and keep the host image clean:

.. code-block:: console

   $ toolbox create
   $ toolbox enter
   $ sudo dnf install gcc make    # inside the toolbox, like a normal Fedora
