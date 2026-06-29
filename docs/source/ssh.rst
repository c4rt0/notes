SSH
===

Config (~/.ssh/config)
----------------------

Define per-host settings so you can just type ``ssh myserver``:

.. code-block:: console

   Host myserver
       HostName 203.0.113.10
       User adam
       Port 2222
       IdentityFile ~/.ssh/id_ed25519

   Host *.internal
       User admin
       ProxyJump bastion           # hop through a jump host automatically

   Host *
       AddKeysToAgent yes
       ServerAliveInterval 60      # keep long-idle sessions alive

Keys and the agent
------------------

.. code-block:: console

   $ ssh-keygen -t ed25519 -C "you@example.com"   # generate a key
   $ ssh-copy-id user@host                         # install your pubkey there
   $ eval "$(ssh-agent -s)" && ssh-add ~/.ssh/id_ed25519
   $ ssh-add -l                                    # list loaded keys
   $ ssh -A user@host                              # forward the agent (use with care)

ProxyJump (bastion hosts)
-------------------------

.. code-block:: console

   $ ssh -J bastion user@internal-host    # one or more comma-separated hops
   # equivalently: "ProxyJump bastion" in ~/.ssh/config

Port forwarding and tunnels
---------------------------

.. code-block:: console

   $ ssh -L 8080:localhost:80 user@host    # local:  your localhost:8080 -> host :80
   $ ssh -R 9000:localhost:3000 user@host  # remote: host :9000 -> your localhost:3000
   $ ssh -D 1080 user@host                 # dynamic SOCKS proxy on :1080
   $ ssh -fNL 5432:db.internal:5432 user@bastion   # background tunnel, no shell

Running commands and copying files
----------------------------------

.. code-block:: console

   $ ssh user@host 'uname -a; uptime'      # run a remote command
   $ scp file user@host:/tmp/              # copy a file
   $ rsync -avz dir/ user@host:/srv/dir/   # sync a directory (resumable)
   $ ssh user@host 'tar czf - /data' | tar xzf -   # stream a dir back

Debugging
---------

.. code-block:: console

   $ ssh -v user@host             # verbose - shows auth methods and key tries
   # On a hung session, type on a fresh line:  ~.  to kill it,  ~C  for a command line
