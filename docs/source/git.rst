Git
===================================

commit
--------------

When running ``git commit`` without a flag, add the ``Co-authored-by`` trailer to credit collaborators:

.. code-block:: console

         Co-authored-by: Adam Piasecki <c4rt0gr4ph3r@redhat.com>

This goes at the bottom of the commit message body, separated by a blank line. GitHub will recognize both authors in the commit.

Squash commits before merging a PR to keep the history clean — one commit per logical change is easier to review and revert:

.. code-block:: console

        $ git rebase -i HEAD~3

This opens an editor where you mark commits to ``squash`` or ``fixup`` into the first one.

rebase workflow
--------------

To rebase your feature branch on top of the latest main:

.. code-block:: console

        $ git fetch origin
        $ git rebase origin/main

If conflicts arise, resolve them file by file, then:

.. code-block:: console

        $ git add <resolved-file>
        $ git rebase --continue

To abort a rebase that went wrong:

.. code-block:: console

        $ git rebase --abort

useful commands
--------------

Show what changed between your branch and main (useful before opening a PR):

.. code-block:: console

        $ git log --oneline origin/main..HEAD
        $ git diff origin/main...HEAD --stat

Amend the last commit message (only if not yet pushed):

.. code-block:: console

        $ git commit --amend

Unstage a file without losing changes:

.. code-block:: console

        $ git restore --staged <file>

View the history of a specific file:

.. code-block:: console

        $ git log --follow -p -- <file>

Search commit messages for a keyword:

.. code-block:: console

        $ git log --grep="iscsi" --oneline
