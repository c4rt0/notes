Fedora Silverblue
=====================


I use Silverblue for a while now - it tooks a few runs to understand and feel the quite unique purpuse of this distro.
It's quite an interesting and impressive aproach to the OS - I deffinitely encourage anyone to try it out. As anything in life ...
You got to try it to know.

If you're acustomed of the stanard DNF or APT package managers - this one is a little different. 
Below you will see commands that helped me along the way to modify, use or configure the system to my liking. 
Obviously it's not the official documentation but a product of annoying errors which had to be logged since memory sometimes dissapoints.
Feel free to reach out in case something's off - I hope you'll find this helpfull.

Commands
------------------------------------------------

Rebase the system from F39 to F40.

.. code-block:: console

    rpm-ostree rebase fedora:fedora/40/x86_64/silverblue

Rebase rpm-ostree itself to the `latest one with quite a low karma at the time <https://bodhi.fedoraproject.org/updates/FEDORA-2024-95b6bafb8b>`_

.. code-block:: console

    rpm-ostree override replace https://bodhi.fedoraproject.org/updates/FEDORA-2024-95b6bafb8b

