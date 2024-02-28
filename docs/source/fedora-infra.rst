Fedora-infra
===================================

Connecting to batcave on fedora-infra to run ansible playbooks
--------------------------------

1. Modify the *~/.ssh/config* file according to `the docs <https://docs.fedoraproject.org/en-US/infra/sysadmin_guide/sshaccess/>`_

.. code-block:: console

    # batcave for FCOS
    Host bastion.fedoraproject.org
    HostName bastion.fedoraproject.org
    User c4rt0
    ProxyCommand none
    ForwardAgent no
    VerifyHostKeyDNS yes
    IdentityFile ~/.ssh/id_ed25519_github
    Host *.iad2.fedoraproject.org *.qa.fedoraproject.org 10.3.160.* 10.3.161.* 10.3.163.* 10.3.165.* 10.3.167.* 10.3.171.* *.vpn.fedoraproject.org
    ProxyJump bastion.fedoraproject.org
    Host batcave
    HostName %h01.iad2.fedoraproject.org
    User c4rt0

This slightly modified version above allows me to run: *ssh batcave* and get connected.

2. Update the `fedora account settings <https://accounts.fedoraproject.org/user/c4rt0/settings/profile/>`_ to use OTP (2-factor authentication)

3. I got sponsored and became a member of two additional groups:
*sysadmin* - Fedora Sysadmin Group
and
*sysadmin-coreos*

4. Once on batcave I first had to pull the fedora-infra repo:

.. code-block:: console

    git clone https://pagure.io/fedora-infra/ansible

Once that was done I ran first playbook with *sudo* and *-C* option:

.. code-block:: console

    sudo rbac-playbook -C openshift-apps/fedora-coreos-pipeline.yml```

from *rbac-playbook -h*:

.. code-block:: console

    -C, --check           don't make any changes; instead, try to predict some of the changes that may occur

Another useful commands I ran while on batcave:

*groups* - displayed the fedora groups I'm a member of.