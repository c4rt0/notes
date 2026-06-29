systemd
=======

Managing services
-----------------

.. code-block:: console

   $ systemctl status sshd            # state, recent logs, main PID
   $ systemctl start sshd             # also stop / restart
   $ systemctl reload sshd            # re-read config without a full restart
   $ systemctl enable --now sshd      # start now AND on every boot
   $ systemctl disable --now sshd
   $ systemctl is-active sshd; systemctl is-enabled sshd
   $ systemctl mask sshd              # fully block it (symlink to /dev/null)

Listing and inspecting units
----------------------------

.. code-block:: console

   $ systemctl list-units --type=service        # running units
   $ systemctl list-units --state=failed        # what is broken
   $ systemctl list-unit-files --state=enabled  # enabled on boot
   $ systemctl cat sshd                         # show the unit file(s)
   $ systemctl show sshd -p MainPID,ActiveState # raw properties
   $ systemctl daemon-reload                    # after editing a unit by hand

Overriding a unit (drop-ins)
----------------------------

Don't edit vendor units in ``/usr/lib``; use a drop-in so changes survive
package updates:

.. code-block:: console

   $ systemctl edit sshd    # creates /etc/systemd/system/sshd.service.d/override.conf

.. code-block:: ini

   [Service]
   Restart=always
   Environment=FOO=bar

Writing a unit
--------------

``/etc/systemd/system/myapp.service``:

.. code-block:: ini

   [Unit]
   Description=My app
   After=network-online.target
   Wants=network-online.target

   [Service]
   ExecStart=/usr/local/bin/myapp
   Restart=on-failure
   User=myapp

   [Install]
   WantedBy=multi-user.target

journalctl
----------

.. code-block:: console

   $ journalctl -u sshd               # logs for one unit
   $ journalctl -u sshd -f            # follow, like tail -f
   $ journalctl -b                    # this boot;  -b -1 = previous boot
   $ journalctl -p err                # priority err and worse
   $ journalctl --since "10 min ago" --until now
   $ journalctl -k                    # kernel messages (dmesg)
   $ journalctl --disk-usage; journalctl --vacuum-time=7d

Timers (a cron replacement)
---------------------------

A ``.timer`` triggers the ``.service`` of the same name. ``backup.timer``:

.. code-block:: ini

   [Unit]
   Description=Nightly backup

   [Timer]
   OnCalendar=*-*-* 02:00:00     # daily at 02:00
   Persistent=true               # run on next boot if a scheduled run was missed

   [Install]
   WantedBy=timers.target

.. code-block:: console

   $ systemctl enable --now backup.timer
   $ systemctl list-timers            # next/last run of every timer

Targets (runlevels)
-------------------

.. code-block:: console

   $ systemctl get-default
   $ systemctl set-default multi-user.target   # boot without a GUI
   $ systemctl isolate multi-user.target        # switch now, no reboot

Boot analysis
-------------

.. code-block:: console

   $ systemd-analyze                  # total boot time
   $ systemd-analyze blame            # slowest units
   $ systemd-analyze critical-chain   # the critical boot path

User units
----------

Run units as your own user (no root) - this is how rootless container quadlets
run:

.. code-block:: console

   $ systemctl --user enable --now myapp.service
   $ loginctl enable-linger $USER     # let user units run without an active login
