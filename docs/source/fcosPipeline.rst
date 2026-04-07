CoreOS pipeline
===================================

This doc page relates mainly to modifying the build configuration of the Jenkins <b>`rhcos-devel`</b> pipeline.

`fedora-coreos-pipeline config <https://github.com/coreos/fedora-coreos-pipeline/blob/main/docs/config.yaml>`_ and also `rhcos-devel-pipecfg <https://gitlab.cee.redhat.com/coreos/rhcos-devel-pipecfg/-/blob/main/config.yaml?ref_type=heads>`_ (accessible via VPN only):

Here's also a `link to my fork <https://gitlab.cee.redhat.com/apiaseck/rhcos-devel-pipecfg>`_ of the above config.

`Red Hat openshift console <https://console-openshift-console.apps.ocp-virt.prod.psi.redhat.com/k8s/ns/rhcos-devel/configmaps/jenkins-config>`_ for rhcos-devel

`Jenkins rhcos devel <https://jenkins-rhcos-devel.apps.ocp-virt.prod.psi.redhat.com/>`_ used for builds

1. Make sure you have access to Jenkins and Openshift console for the `rhcos-devel`

.. todo:: Insert appropiate link to the file where allowed users are listed

Jenkins Error message
--------------

Example of a ``kola testiso`` failure in the pipeline:

.. code-block:: console

        [2024-01-08T18:22:59.748Z] Running test: iso-offline-install-iscsi.bios
        [2024-01-08T18:28:36.181Z] FAIL: iso-offline-install-iscsi.bios (5m35.266s)
        [2024-01-08T18:28:36.182Z]     QEMU exited; timed out waiting for completion
        ......
        [2024-01-08T18:43:43.997Z] Error: harness: test suite failed
        [2024-01-08T18:43:43.997Z] 2024-01-08T18:43:39Z cli: harness: test suite failed
        [2024-01-08T18:43:43.997Z] failed to execute cmd-kola: exit status 1
        script returned exit code 1

The ``QEMU exited; timed out waiting for completion`` message means the VM didn't finish the install within the timeout window.

Debugging steps
^^^^^^^^^^^^^^^

1. Check the full Jenkins console output for earlier errors that may have caused the timeout.
2. Reproduce locally by running the failing test inside a cosa container:

.. code-block:: console

        $ cosa kola testiso -S iso-offline-install-iscsi --qemu-console-log

3. Inspect the console log for boot failures, missing network config, or iSCSI target issues.
4. If the test passes locally, the failure may be resource-related (CI memory/CPU constraints). Check if other tests in the same run also timed out.