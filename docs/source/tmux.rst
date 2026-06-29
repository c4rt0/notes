tmux
====

A terminal multiplexer: persistent sessions you can detach from and reattach to
(survives SSH drops), split into panes and windows. The default **prefix** is
``Ctrl-b`` (written ``C-b`` below) - press it, release, then the key.

Sessions
--------

.. code-block:: console

   $ tmux                       # start a session
   $ tmux new -s work           # named session
   $ tmux ls                    # list sessions
   $ tmux attach -t work        # reattach  (tmux a -t work)
   $ tmux kill-session -t work
   # inside:  C-b d  detach   |  C-b $  rename session  |  C-b s  switch session

Windows (tabs)
--------------

.. code-block:: console

   # C-b c        new window
   # C-b ,        rename window
   # C-b n / p    next / previous window
   # C-b 0..9     jump to a window by number
   # C-b w        list / choose windows
   # C-b &        kill window

Panes (splits)
--------------

.. code-block:: console

   # C-b %        split left | right
   # C-b "        split top / bottom
   # C-b <arrow>  move between panes
   # C-b o        cycle panes      |  C-b z  zoom / unzoom a pane
   # C-b x        kill pane        |  C-b space  cycle layouts

Copy mode and scrolling
-----------------------

.. code-block:: console

   # C-b [        enter copy mode (scroll with arrows / PgUp, search with /)
   #   Space      start selection,  Enter  copy,  q  quit
   # C-b ]        paste

Config (~/.tmux.conf)
---------------------

.. code-block:: console

   set -g mouse on                 # mouse scroll / select / resize
   set -g base-index 1             # windows start at 1, not 0
   set -g history-limit 100000
   bind r source-file ~/.tmux.conf \; display "reloaded"   # C-b r to reload

Scripting a layout
------------------

Build a dev environment non-interactively:

.. code-block:: console

   tmux new -d -s dev
   tmux send-keys -t dev 'cd ~/project && vim' Enter
   tmux split-window -h -t dev
   tmux send-keys -t dev 'npm run dev' Enter
   tmux attach -t dev
