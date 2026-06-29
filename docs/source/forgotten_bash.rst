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

history shortcuts
-----------------

The shell remembers your last commands and their arguments - stop retyping:

.. code-block:: console

   $ sudo !!                 # re-run the LAST command, prefixed with sudo
   $ mkdir newdir && cd !$   # !$ = last argument of the previous command
   $ cp file.txt /backup/ && cd $_   # $_ also = last arg (works mid-script)
   $ ^typo^fixed             # re-run last command, replacing "typo" with "fixed"
   $ !curl                   # re-run the most recent command starting with "curl"
   # Ctrl+R   : reverse-search history as you type
   # Ctrl+X Ctrl+E : open the current command line in $EDITOR

brace expansion
---------------

The shell expands ``{...}`` before running anything - great for backups and bulk
creation:

.. code-block:: console

   $ cp config.yaml{,.bak}              # -> cp config.yaml config.yaml.bak
   $ mv report-{2023,2024}.csv          # rename report-2023.csv -> report-2024.csv
   $ mkdir -p project/{src,test,docs}   # create three directories at once
   $ echo {1..5}                        # 1 2 3 4 5
   $ touch file-{01..10}.txt            # zero-padded ranges work too

process substitution
--------------------

``<(cmd)`` makes a command's output look like a file - perfect for diffing two
commands without temp files:

.. code-block:: console

   $ diff <(sort a.txt) <(sort b.txt)
   $ diff <(rpm -qa | sort) <(ssh host 'rpm -qa | sort')   # compare package sets
   $ comm -12 <(sort a) <(sort b)       # lines common to both

text wrangling
--------------

.. code-block:: console

   $ tac file.log                       # cat in reverse (last line first)
   $ column -t -s, data.csv             # align CSV into neat columns
   $ paste -d, col1.txt col2.txt        # join files side by side
   $ shuf -n1 list.txt                  # pick a random line

xargs
-----

Build and run commands from stdin - ``{}`` is each item, ``-P`` runs in parallel:

.. code-block:: console

   $ find . -name '*.tmp' | xargs rm
   $ ls *.png | xargs -I{} mv {} images/{}
   $ cat urls.txt | xargs -P4 -n1 curl -O   # 4 downloads at a time

tee
---

Write to a file and stdout at once - and the classic trick for writing a
root-owned file without a root shell:

.. code-block:: console

   $ make 2>&1 | tee build.log
   $ echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-fwd.conf

small but handy
---------------

.. code-block:: console

   $ timeout 5 ping example.com         # give up after 5 seconds
   $ yes | dnf install foo              # auto-answer "y" to every prompt
   $ seq 1 2 10                         # 1 3 5 7 9  (start step end)
   $ mktemp -d                          # create a safe temp directory
   $ disown -h %1                       # keep a background job alive after logout

trap - clean up on exit
-----------------------

Run cleanup whenever a script exits, even on error or Ctrl+C:

.. code-block:: console

   tmp=$(mktemp -d)
   trap 'rm -rf "$tmp"' EXIT            # always removes the temp dir on exit

watching progress
-----------------

``pv`` (pipe viewer) shows a live progress bar, throughput and ETA for anything
flowing through a pipe (``dnf install -y pv``):

.. code-block:: console

   $ pv big.iso | gzip > big.gz                 # progress bar + ETA while compressing
   $ pv /dev/zero > /dev/null                   # measure raw throughput
   $ tar czf - /data | pv | ssh host 'cat > data.tgz'   # progress over the network

For a quick spinner in your own script while a slow job runs:

.. code-block:: console

   slow-task & pid=$!
   while kill -0 $pid 2>/dev/null; do
     for c in '|' '/' '-' '\\'; do printf "\r%s working..." "$c"; sleep 0.1; done
   done; printf "\rdone        \n"

timers and stopwatches
----------------------

.. code-block:: console

   $ time some-command            # how long a command took (real/user/sys)
   $ time read                    # crude stopwatch - press Enter to stop
   $ sleep 25m && tput bel        # a 25-min pomodoro; rings the terminal bell
   $ for i in {5..1}; do printf "\r%2ds " "$i"; sleep 1; done; echo "go!"   # countdown
   $ tty-clock -c -C4             # a centered terminal clock (needs tty-clock)

terminal toys
-------------

Pure fun. Most are separate packages - install with ``dnf install <name>``:

.. code-block:: console

   $ cmatrix                      # the Matrix "digital rain"
   $ sl                           # a steam locomotive chuffs by when you mistype "ls"
   $ fortune | cowsay | lolcat    # a random quote, from a cow, in rainbow
   $ figlet "HELLO"               # big ASCII-art banner text
   $ pipes.sh                     # animated pipes screensaver
   $ cbonsai -l                   # a slowly growing bonsai tree
   $ asciiquarium                 # an ASCII aquarium
   $ nyancat                      # the nyan cat, forever
   $ neofetch                     # system info next to your distro logo
