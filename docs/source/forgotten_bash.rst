Forgotten bash commands
===================================

There are a few usefull bash commands I came across during my software career, that has a special meaning and should be used more often. I guess everyone has theyre own workflow, but mine works well with the few forgotten ones below:

pushd
-----

Directory stack tool used in shell scripting 

.. code-block:: console

   pushd 

popd
----

Directory stack tool used in shell scripting 

.. code-block:: console

   popd

curl cheat.sh
-------------

Summarizes man pages for particular command, example:

.. code-block:: console

   curl cheat.sh/vim*

Here I even went a step further and edited my ~/.zshrc with a new alias:

.. code-block:: console

   alias ch = 'curl cheat.sh/${0}'*

This way I can simply type in 'ch vim' and look up docs for anything much quicker

progress
--------

Shows progress while copying large files in terminal

.. code-block:: console

   cp file.a /directory & progress -mp $!

.. code-block:: console

   sudo dnf install progress -y

watch
-----

Watches if file `console.txt` exists and sends signal if it does

.. code-block:: console

   watch -n 0.1 ls console.txt
