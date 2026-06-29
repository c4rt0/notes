GPG & SSH keys
==============

Key management, backup, and YubiKey notes.

Managing multiple identities
----------------------------

It's common to run separate keys per identity (e.g. a personal GitHub key and a
work key), each with its own email and GPG key. Useful patterns:

- Sign per-repo with the right key via a repo-local
  ``git config user.signingkey <fingerprint>`` (and ``commit.gpgsign true``)
  instead of flipping the global identity.
- When your system username differs from a service username (e.g. Fedora's FAS),
  tools like ``fedpkg`` may need a ``gitbaseurl`` override in
  ``~/.config/rpkg/fedpkg.conf``.
- Register SSH keys for the relevant service (e.g.
  https://accounts.fedoraproject.org/).

Backing up GPG keys
-------------------

**Always back up before any destructive operation** (e.g. ``keytocard``):

.. code-block:: bash

        # Full GPG directory backup
        cp -r ~/.gnupg ~/.gnupg-backup-$(date +%Y%m%d)

        # Export a private key separately
        gpg --armor --export-secret-keys <your-email> > ~/key-secret.asc

Store backups on an encrypted USB drive. ``keytocard`` **deletes** the private
key from disk - it's a one-way move, not a copy. Restore if needed:

.. code-block:: bash

        rm -rf ~/.gnupg
        cp -r ~/.gnupg-backup-YYYYMMDD ~/.gnupg

YubiKey architecture
--------------------

GPG, the touch buttons, and FIDO2 are independent applets on separate hardware
interfaces - loading GPG keys does **not** affect button behavior.

.. list-table::
   :header-rows: 1

   * - Feature
     - Interface
     - Notes
   * - Short press (OTP)
     - Slot 1
     - Yubico OTP or static password
   * - Long press
     - Slot 2
     - HOTP, challenge-response, or static password
   * - GPG keys
     - OpenPGP smart-card applet
     - independent (smart-card interface)
   * - FIDO2 / U2F
     - FIDO applet
     - independent

Key storage options
-------------------

.. list-table::
   :header-rows: 1

   * - Method
     - Strength
     - Use case
   * - YubiKey
     - keys never on disk, portable
     - daily use across machines
   * - Encrypted USB
     - offline, air-gapped
     - disaster-recovery backup
   * - Password manager
     - encrypted, shareable
     - recovery, secrets sharing
   * - ``pass`` (password-store)
     - GPG-encrypted git repo
     - CLI-friendly (but circular dep on GPG)
   * - Per-machine SSH keys
     - revoke individually
     - one key per machine

Recommended setup: YubiKey for daily use + encrypted USB backup.
