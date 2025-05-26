CoreOS
===================================

I keep forgetting stuff, so here I will be placing the usefull CoreOS related commands.

building rhcos
--------------

A simple recipe to build latest RHCOS image:
source: https://github.com/openshift/os/blob/master/docs/development-rhcos.md

.. code-block:: console

        $ mkdir rhcos
        $ cd rhcos
        $ mkdir rhcos-4.15
        $ cd rhcos-4.15
        $ cosa shell # provided your cosa environment is set up - example below under `cosa func`
        $ export COREOS_ASSEMBLER_ADD_CERTS='y'
        $ export RHCOS_REPO="https://your.rhcos.repo.example.com/coreos/redhat-coreos.git"
        $ cosa init --yumrepos "${RHCOS_REPO}" --branch release-4.15 https://github.com/openshift/os.git
        $ cosa fetch
        $ cosa build

Final result:

.. code-block:: console

        $ ls builds/
        415.92.202312121624-0  builds.json  latest
        
        $ ls builds/latest/x86_64/
        commitmeta.json                     manifest.json                        rhcos-415.92.202312121624-0-ostree.x86_64-manifest.json
        coreos-assembler-config-git.json    manifest-lock.generated.x86_64.json  rhcos-415.92.202312121624-0-ostree.x86_64.ociarchive
        coreos-assembler-config.tar.gz      meta.json                            rhcos-415.92.202312121624-0-qemu.x86_64.qcow2
        coreos-assembler-yumrepos-git.json  ostree-commit-object


cosa func
---------

.. code-block:: console

        cosa() {
           env | grep COREOS_ASSEMBLER
           local -r COREOS_ASSEMBLER_CONTAINER_LATEST="quay.io/coreos-assembler/coreos-assembler:latest"
           if [[ -z ${COREOS_ASSEMBLER_CONTAINER} ]] && $(podman image exists ${COREOS_ASSEMBLER_CONTAINER_LATEST}); then
               local -r cosa_build_date_str="$(podman inspect -f "{{.Created}}" ${COREOS_ASSEMBLER_CONTAINER_LATEST} | awk '{print $1}')"
               local -r cosa_build_date="$(date -d ${cosa_build_date_str} +%s)"
               if [[ $(date +%s) -ge $((cosa_build_date + 60*60*24*7)) ]] ; then
                 echo -e "\e[0;33m----" >&2
                 echo "COSA is outdated." >&2
                 # echo "podman pull ${COREOS_ASSEMBLER_CONTAINER_LATEST}" >&2
                 # echo -e "----\e[0m" >&2
                 pp
                 sleep 10
               fi
           fi
           set -x
           podman run --rm -ti --security-opt=label=disable --privileged                                    \
                      --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap=1001:1001:64536                          \
                      -v=${PWD}:/srv/ --device=/dev/kvm --device=/dev/fuse                                  \
                      --tmpfs=/tmp -v=/var/tmp:/var/tmp --name=cosa                                         \
                      ${COREOS_ASSEMBLER_CONFIG_GIT:+-v=$COREOS_ASSEMBLER_CONFIG_GIT:/srv/src/config/:ro}   \
                      ${COREOS_ASSEMBLER_GIT:+-v=$COREOS_ASSEMBLER_GIT/src/:/usr/lib/coreos-assembler/:ro}  \
                      ${COREOS_ASSEMBLER_ADD_CERTS:+-v=/etc/pki/ca-trust:/etc/pki/ca-trust:ro}              \
                      ${COREOS_ASSEMBLER_CONTAINER_RUNTIME_ARGS}                                            \
                      ${COREOS_ASSEMBLER_CONTAINER:-$COREOS_ASSEMBLER_CONTAINER_LATEST} "$@"
           rc=$?; set +x; return $rc
        }


# Playing arround with customisation:

Running the iscisi
------------------

.. code-block:: console

        adamsky@laptop Work/coreos-assembler (pr/testiscsi %) » podman run --rm -ti --security-opt=label=disable --privileged \
        --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap=1001:1001:64536 \
        -v=${PWD}:/srv/ --device=/dev/kvm --device=/dev/fuse \
        --tmpfs=/tmp -v=/var/tmp:/var/tmp \
        -v=/home/adamsky/Work/coreos-assembler-hacking/:/srv/fcos \
        --name=cosa quay.io/coreos-assembler/coreos-assembler:latest shell

        [coreos-assembler]$ cd fcos
        [coreos-assembler]$ pwd
        /srv/fcos
        [coreos-assembler]$ ls
        builds  cache  overrides  src  tmp
        [coreos-assembler]$ ../mantle/build kola
        Building kola
        [coreos-assembler]$ ../bin/kola testiso -S iso-install-iscsi


===================================

cosa shell
-----------

.. code-block:: console

        ./mantle/build kola
        ./bin/kola list | grep coreos.unique.boot.failure
        ./bin/kola run -b fcos --qemu-image fedora-coreos-38.20230918.dev.0-qemu.x86_64.qcow2 coreos.unique.boot.failure
        [coreos-assembler]$ ./mantle/build kola
        Building kola
        [coreos-assembler]$ ./bin/kola run -b fcos --qemu-image fedora-coreos-38.20230918.dev.0-qemu.x86_64.qcow2 coreos.unique.boot.failure

        podman run --rm -ti --security-opt=label=disable --privileged --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap=1001:1001:64536 -v=${PWD}:/srv/ --device=/dev/kvm --device=/dev/fuse --tmpfs=/tmp -v=/var/tmp:/var/tmp -v=/home/adamsky/Work/coreos-assembler-hacking/:/srv/fcos --name=cosa quay.io/coreos-assembler/coreos-assembler:latest shell


cosa spawn and zincati
----------


Here's how to spawn a new cosa vm on aws, while having a direct access to it's cli:

.. code-block:: console

        10253  mkdir cosa_test
        10254  cd cosa_test
        10255  cosa init https://github.com/coreos/fedora-coreos-config
        10256  cosa buildfetch --stream stable --artifact qemu
        10257  cp ../cosa_test/cred .
        10258  ls
        10259  cosa kola spawn -b fcos --stream=stable -p=aws --aws-region=us-east-1 --aws-type=i3.large --aws-credentials-file cred


Format of the credentials file:
-------------------------------

.. code-block:: console

        [default]
        aws_access_key_id=ABRACADABRA
        aws_secret_access_key=50m35ecr3t4w5k3y
        region = us-east-1
        output = text


Checking zincati logs
---------------------

.. code-block:: console

        [bound] -bash-5.2$ journalctl -u zincati
        May 22 10:32:48 ip-172-31-41-244 systemd[1]: Starting zincati.service - Zincati Update Agent...
        May 22 10:32:48 ip-172-31-41-244 zincati[1910]: [INFO  zincati::cli::agent] starting update agent (zincati 0.0.30)
        May 22 10:32:49 ip-172-31-41-244 zincati[1910]: [INFO  zincati::cincinnati] Cincinnati service: https://updates.coreos.fedoraproject.org
        May 22 10:32:49 ip-172-31-41-244 zincati[1910]: [INFO  zincati::cli::agent] agent running on node '7bd11cadfe1a457cbfebe3118fae9a56', in update group 'default'
        May 22 10:32:49 ip-172-31-41-244 zincati[1910]: [WARN  zincati::update_agent::actor] initialization complete, auto-updates logic disabled by configuration
        May 22 10:32:49 ip-172-31-41-244 systemd[1]: Started zincati.service - Zincati Update Agent.


When however updates are [automatically disabled](https://github.com/coreos/coreos-assembler/blob/6ec2120eca938b4678a9c683a567dd562a73b7b7/mantle/platform/cluster.go#L271-L272) 
look for the *disable-auto-updates.toml in:

.. code-block:: console
        
        [bound] -bash-5.2$ pwd
        /etc/zincati/config.d
        [bound] -bash-5.2$ cat 90-disable-auto-updates.toml 
        [updates]
                enabled = false


After the above is found, remove it and restart zincati (it now should work fine):

.. code-block:: console

        [bound] -bash-5.2$ systemctl restart zincati
        [bound] -bash-5.2$ journalctl -u zincati
        May 22 12:05:29 ip-172-31-24-128 systemd[1]: Starting zincati.service - Zincati Update Agent...
        May 22 12:05:29 ip-172-31-24-128 zincati[1908]: [INFO  zincati::cli::agent] starting update agent (zincati 0.0.30)
        May 22 12:05:30 ip-172-31-24-128 zincati[1908]: [INFO  zincati::cincinnati] Cincinnati service: https://updates.coreos.fedoraproject.org
        May 22 12:05:30 ip-172-31-24-128 zincati[1908]: [INFO  zincati::cli::agent] agent running on node '9450a569670a4d5cbf5495b6ee33dc7b', in update group 'default'
        May 22 12:05:30 ip-172-31-24-128 zincati[1908]: [WARN  zincati::update_agent::actor] initialization complete, auto-updates logic disabled by configuration
        May 22 12:05:30 ip-172-31-24-128 systemd[1]: Started zincati.service - Zincati Update Agent.
        May 22 12:07:11 ip-172-31-24-128 systemd[1]: Stopping zincati.service - Zincati Update Agent...
        May 22 12:07:11 ip-172-31-24-128 systemd[1]: zincati.service: Deactivated successfully.
        May 22 12:07:11 ip-172-31-24-128 systemd[1]: Stopped zincati.service - Zincati Update Agent.
        May 22 12:07:11 ip-172-31-24-128 systemd[1]: Starting zincati.service - Zincati Update Agent...
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::cli::agent] starting update agent (zincati 0.0.30)
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::cincinnati] Cincinnati service: https://updates.coreos.fedoraproject.org
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::cli::agent] agent running on node '9450a569670a4d5cbf5495b6ee33dc7b', in update group 'default'
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::update_agent::actor] registering as the update driver for rpm-ostree
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::update_agent::actor] initialization complete, auto-updates logic enabled
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::strategy] update strategy: immediate
        May 22 12:07:12 ip-172-31-24-128 systemd[1]: Started zincati.service - Zincati Update Agent.
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::update_agent::actor] reached steady state, periodically polling for updates
        May 22 12:07:12 ip-172-31-24-128 zincati[2276]: [INFO  zincati::cincinnati] current release detected as not a dead-end
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
