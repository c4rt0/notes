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

Testing own stuff
----------

Running the iscisi stuff

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

./mantle/build kola
./bin/kola list | grep coreos.unique.boot.failure
./bin/kola run -b fcos --qemu-image fedora-coreos-38.20230918.dev.0-qemu.x86_64.qcow2 coreos.unique.boot.failure

[coreos-assembler]$ ./mantle/build kola
Building kola
[coreos-assembler]$ ./bin/kola run -b fcos --qemu-image fedora-coreos-38.20230918.dev.0-qemu.x86_64.qcow2 coreos.unique.boot.failure

podman run --rm -ti --security-opt=label=disable --privileged --uidmap=1000:0:1 --uidmap=0:1:1000 --uidmap=1001:1001:64536 -v=${PWD}:/srv/ --device=/dev/kvm --device=/dev/fuse --tmpfs=/tmp -v=/var/tmp:/var/tmp -v=/home/adamsky/Work/coreos-assembler-hacking/:/srv/fcos --name=cosa quay.io/coreos-assembler/coreos-assembler:latest shell

cosa kola 

////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
