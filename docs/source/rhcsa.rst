RHCSA preparation
===================================

Practice questions grouped by exam objective. Each question includes a scenario and solution.


Networking
----------

**Question 1:** Configure a static network connection.

You have been provided a virtual box named serverX.example.com (X is your domain number).
Password for the virtual machine is ``redhat123``.

* serverX.example.com has IP: ``172.25.X.11/255.255.255.0``
* Gateway: ``172.25.254.254``
* DNS domain: ``example.com`` with IP ``172.25.254.254``

.. code-block:: console

        $ hostname                          # check domain number to determine X
        $ nmcli dev status                  # verify which device to configure
        $ nmcli con add con-name lab \
            ifname eth0 type ethernet \
            ip4 172.25.X.11/24 \
            gw4 172.25.254.254
        $ nmcli con mod lab ipv4.dns 172.25.254.254
        $ nmcli con up lab

Alternatively, use the TUI:

.. code-block:: console

        $ nmtui


Users and groups
-----------------

**Question 2:** Create users with specific group memberships.

Create a group ``sysadmins``. Create users ``alice`` and ``bob``, both members of ``sysadmins`` as a supplementary group. Set ``alice``'s account to expire on 2026-12-31.

.. code-block:: console

        $ groupadd sysadmins
        $ useradd -G sysadmins alice
        $ useradd -G sysadmins bob
        $ chage -E 2026-12-31 alice
        $ chage -l alice                    # verify expiry date

**Question 3:** Set a user's default shell and password policy.

Set ``bob``'s default shell to ``/bin/zsh``. Require ``bob`` to change his password every 90 days with a 7-day warning.

.. code-block:: console

        $ usermod -s /bin/zsh bob
        $ chage -M 90 -W 7 bob
        $ chage -l bob


Permissions and ACLs
---------------------

**Question 4:** Configure group-shared directory with sticky bit.

Create ``/shared/engineering``. The ``sysadmins`` group should own it. New files inside must inherit the group. Users should not be able to delete each other's files.

.. code-block:: console

        $ mkdir -p /shared/engineering
        $ chown :sysadmins /shared/engineering
        $ chmod 2770 /shared/engineering    # setgid ensures group inheritance
        $ chmod +t /shared/engineering      # sticky bit prevents deletion by others
        $ ls -ld /shared/engineering        # verify: drwxrws--T

**Question 5:** Use ACLs to grant specific user access.

Grant user ``bob`` read-only access to ``/data/reports`` without changing the base permissions.

.. code-block:: console

        $ setfacl -m u:bob:rx /data/reports
        $ getfacl /data/reports             # verify ACL entry


SELinux
--------

**Question 6:** Fix a web server serving content from a non-default directory.

Apache is configured to serve from ``/custom/www`` but returns 403 Forbidden.

.. code-block:: console

        $ ls -Z /custom/www                 # check current context
        $ semanage fcontext -a -t httpd_sys_content_t "/custom/www(/.*)?"
        $ restorecon -Rv /custom/www
        $ curl http://localhost               # verify it works

**Question 7:** Diagnose and fix an SELinux boolean issue.

``httpd`` cannot connect to a database on another host.

.. code-block:: console

        $ sealert -a /var/log/audit/audit.log   # check for AVC denials
        $ getsebool httpd_can_network_connect_db
        $ setsebool -P httpd_can_network_connect_db on


Storage and LVM
----------------

**Question 8:** Create a logical volume and mount it persistently.

Create a 500M logical volume named ``data-lv`` in volume group ``data-vg`` using ``/dev/sdb``. Format as XFS and mount at ``/mnt/data``.

.. code-block:: console

        $ pvcreate /dev/sdb
        $ vgcreate data-vg /dev/sdb
        $ lvcreate -n data-lv -L 500M data-vg
        $ mkfs.xfs /dev/data-vg/data-lv
        $ mkdir /mnt/data
        $ echo "/dev/data-vg/data-lv /mnt/data xfs defaults 0 0" >> /etc/fstab
        $ mount -a
        $ df -h /mnt/data                   # verify

**Question 9:** Extend a logical volume while mounted.

Grow ``data-lv`` by 200M.

.. code-block:: console

        $ lvextend -L +200M /dev/data-vg/data-lv
        $ xfs_growfs /mnt/data              # XFS: grow the filesystem
        $ df -h /mnt/data                   # verify new size

For ext4, use ``resize2fs /dev/data-vg/data-lv`` instead of ``xfs_growfs``.


Scheduling tasks
-----------------

**Question 10:** Schedule a recurring job with cron and a one-time job with at.

Run ``/usr/local/bin/cleanup.sh`` every weekday at 2:30 AM as root. Also schedule a one-time reboot at 23:00 tonight.

.. code-block:: console

        $ crontab -e
        # add: 30 2 * * 1-5 /usr/local/bin/cleanup.sh

        $ echo "systemctl reboot" | at 23:00


Containers (podman)
--------------------

**Question 11:** Run a container as a systemd service using quadlets.

Run an nginx container publishing port 8080, configured to start on boot as a rootless user service.

.. code-block:: console

        $ mkdir -p ~/.config/containers/systemd
        $ cat > ~/.config/containers/systemd/nginx.container << 'EOF'
        [Unit]
        Description=Nginx web server

        [Container]
        Image=docker.io/library/nginx
        PublishPort=8080:80

        [Service]
        Restart=always

        [Install]
        WantedBy=default.target
        EOF

        $ systemctl --user daemon-reload
        $ systemctl --user start nginx
        $ systemctl --user enable nginx
        $ curl http://localhost:8080


Troubleshooting boot
---------------------

**Question 12:** Reset the root password from the boot loader.

.. code-block:: console

        1. Reboot and interrupt GRUB by pressing 'e'
        2. Find the line starting with 'linux' and append: rd.break
        3. Press Ctrl+X to boot
        4. At the switch_root prompt:
        $ mount -o remount,rw /sysroot
        $ chroot /sysroot
        $ passwd root
        $ touch /.autorelabel              # required for SELinux
        $ exit
        $ exit                             # system reboots
