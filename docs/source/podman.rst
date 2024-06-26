Podman
===================================

Better, deamonless docket... now, let's dive into:

Quadlets
--------------

It started with Alexander Larsson's `idea <https://blogs.gnome.org/alexl/2021/10/12/quadlet-an-easier-way-to-run-system-containers/>`_ who explains the Quadlet, as a `systemd generator <https://www.freedesktop.org/software/systemd/man/systemd.generator.html>`_ that takes a container description and automatically generates a systemd service file from it.

The container description is in the systemd unit file format and describes how you want to run the container (i.e. what image, which ports exposed, etc), as well as standard systemd options, like dependencies. However, it doesn’t need to bother with technical details about how a container gets created or how it integrates with systemd, which makes the file much easier to understand and maintain.

This is easiest demonstrated by an example:

.. code-block:: console

        [Unit]
        Description=Redis container

        [Container]
        Image=docker.io/redis
        PublishPort=6379:6379
        User=999

        [Service]
        Restart=always

        [Install]
        WantedBy=local.target


If you install the above in a file called /etc/containers/systemd/redis.container (or /usr/share/containers/systemd/redis.container) then, during boot (and at systemctl daemon-reload time), this is used to generate the file /run/systemd/generator/redis.service, which is then made available as a regular service.

To get a feeling for this, the above container file generates the following service file:


.. code-block:: console

        # Automatically generated by quadlet-generator
        [Unit]
        Description=Redis container
        RequiresMountsFor=%t/containers
        SourcePath=/etc/containers/systemd/redis.container

        [X-Container]
        Image=docker.io/redis
        PublishPort=6379:6379
        User=999

        [Service]
        Restart=always
        Environment=PODMAN_SYSTEMD_UNIT=%n
        KillMode=mixed
        ExecStartPre=-rm -f %t/%N.cid
        ExecStopPost=-/usr/bin/podman rm -f -i --cidfile=%t/%N.cid
        ExecStopPost=-rm -f %t/%N.cid
        Delegate=yes
        Type=notify
        NotifyAccess=all
        SyslogIdentifier=%N
        ExecStart=/usr/bin/podman run --name=systemd-%N --cidfile=%t/%N.cid --replace --rm -d --log-driver journald --pull=never --runtime /usr/bin/crun --cgroups=split --tz=local --init --sdnotify=conmon --security-opt=no-new-privileges --cap-drop=all --mount type=tmpfs,tmpfs-size=512M,destination=/tmp --user 999 --uidmap 999:999:1 --uidmap 0:0:1 --uidmap 1:362144:998 --uidmap 1000:363142:64538 --gidmap 0:0:1 --gidmap 1:362144:65536 -p=6379:6379 docker.io/redis

        [Install]
        WantedBy=local.target


Once started it looks like a regular service:


.. code-block:: console

        ● redis.service - Redis container
        Loaded: loaded (/etc/containers/systemd/redis.container; generated)
        Active: active (running) since Tue 2021-10-12 12:34:14; 1s ago
        Main PID: 1559371 (conmon)
        Tasks: 8 (limit: 38373)
        Memory: 32.0M
        CPU: 387ms
        CGroup: /system.slice/redis.service
        ├─container
        │ ├─1559375 /dev/init -- docker-entrypoint.sh redis-server
        │ └─1559489 "redis-server *:6379"
        └─supervisor
          └─1559371 /usr/bin/conmon --api-version 1 -c 24184463a9>


In practice you don’t need to care about the generated file, all you need to maintain is the container file. In fact, over time as podman/systemd integration is improved it may generate slightly different files to take advantage of the new features.

In addition to being easier to understand, quadlet comes with a set of defaults for how the container is run that better fit the usecase of running system services. For example, it defaults to running without any capabilities, it has a basic init process in the container, it uses the journal log driver, and it sets up the cgroups in a mode that best matches what systemd needs.
