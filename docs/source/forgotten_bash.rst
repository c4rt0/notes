Forgotten bash commands
=======================

A few bash commands with a special meaning that deserve more use.

pushd / popd - the directory stack
----------------------------------

Instead of ``cd``-ing back and forth, push directories onto a stack and pop back
out. ``dirs -v`` shows the stack with indices.

.. code-block:: console

   $ pushd /var/log      # cd to /var/log AND remember where you were
   $ pushd /etc          # now on /etc; stack: /etc /var/log ~
   $ dirs -v             # list the stack
   $ popd                # pop the top, back to /var/log
   $ cd -                # simpler: toggle between the last two dirs

curl cheat.sh
-------------

``cheat.sh`` returns concise, example-first help for a command:

.. code-block:: console

   $ curl cheat.sh/tar           # cheatsheet for tar
   $ curl cheat.sh/tar/extract   # narrow to a sub-topic

Handy alias (see also :doc:`zshrc`):

.. code-block:: console

   alias ch='curl cheat.sh/$0'   # then just: ch tar

progress
--------

Show progress for a running ``cp``/``mv``/``dd`` (install with
``dnf install -y progress``):

.. code-block:: console

   $ cp big.iso /mnt/ & progress -mp $!   # -m monitor, -p the just-started PID

watch
-----

Re-run a command every N seconds and show the latest output - good for watching
a file appear or a value change:

.. code-block:: console

   $ watch -n 0.1 ls console.txt     # poll every 0.1s
   $ watch -d 'rpm-ostree status'    # -d highlights what changed between runs
