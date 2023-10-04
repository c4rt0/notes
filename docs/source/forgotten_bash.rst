Forgotten bash commands
===================================

There are a few usefull bash commands I came across during my software career, that has a special meaning and should be used more often. I guess everyone has theyre own workflow, but mine works well with the few forgotten ones below:

pushd
-----
Directory stack tool used in shell scripting 

.. code-block:: console
        pushd 

*
popd
----
Directory stack tool used in shell scripting 

.. code-block:: console
        popd

*
curl cheat.sh
-------------

Summarizes man pages for particular command, example:

.. code-block:: console
        curl cheat.sh/vim

*
progress
--------

Shows progress while copying large files in terminal
.. code-block:: console
        cp file.a /directory & progress -mp $!

Note. Requires:

.. code-block:: console
        sudo dnf install progress -y

To be continued...
