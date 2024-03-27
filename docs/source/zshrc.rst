.zshrc & .bashrc modifications
===================================

aliases
-------

Current aliases of my `.zshrc` file I use most:

(To easily edit your config run: `nano ~/.zshrc` or `nano ~/.bashrc`. If you know VIM, you certainly don't need this paragraph.)

.. code-block:: console

        alias butane='podman run --rm --interactive       \
                      --security-opt label=disable        \
                      --volume ${PWD}:/pwd --workdir /pwd \
                      quay.io/coreos/butane:release'

        alias coreos-installer='podman run --pull=always            \
                                --rm --interactive                  \
                                --security-opt label=disable        \
                                --volume ${PWD}:/pwd --workdir /pwd \
                                quay.io/coreos/coreos-installer:release'

        alias ignition-validate='podman run --rm --interactive       \
                                 --security-opt label=disable        \
                                 --volume ${PWD}:/pwd --workdir /pwd \
                                 quay.io/coreos/ignition-validate:release'

        alias ch='curl cheat.sh/$0'
        alias ga='git add .'
        alias main='python main.py'
        alias cdw='cd ~/Work'
        alias cdc='cd ~/Code'
        alias cdf='cd ~/Work/fcos'
        alias cdr='cd ~/Work/rhcos'
        alias cim='vim'
        alias pp='podman pull quay.io/coreos-assembler/coreos-assembler:latest'
        alias teb='toolbox enter banana'
        alias ru='rpm-ostree update'
        alias rs='rpm-ostree status'
        alias sr='systemctl reboot'
        alias ssh-show="ssh-agent sh -c 'ssh-add; ssh-add -L'"
        alias path="echo $PATH | tr \":\" \"\n\""
        alias fcos_dir="/var/home/adamsky/Work/fcos/builds/latest/x86_64"

