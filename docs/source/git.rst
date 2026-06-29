Git
===================================

commit
--------------

When running ``git commit`` without a flag, add the ``Co-authored-by`` trailer to credit collaborators:

.. code-block:: console

         Co-authored-by: Collaborator Name <collaborator@example.com>

This goes at the bottom of the commit message body, separated by a blank line. GitHub will recognize both authors in the commit.

Squash commits before merging a PR to keep the history clean - one commit per logical change is easier to review and revert:

.. code-block:: console

        $ git rebase -i HEAD~3

This opens an editor where you mark commits to ``squash`` or ``fixup`` into the first one.

rebase workflow
---------------

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
---------------

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

comparing branches: ``..`` vs ``...``
-------------------------------------

``git diff A..B`` shows the difference between two points - and what ``A`` and
``B`` are changes the meaning completely:

.. code-block:: console

        $ git diff origin/main..HEAD       # everything your branch adds vs main (before opening a PR)
        $ git diff origin/my-branch..HEAD  # what changed since your last push (before force-pushing)
        $ git diff main..my-branch         # two local branches

Double dot is a direct diff between the two refs. **Triple dot** diffs ``B``
against the *common ancestor* of ``A`` and ``B`` - handy for ``git log`` to list
only the commits on your branch, even if main has moved on:

.. code-block:: console

        $ git log origin/main...HEAD --oneline

For ``git diff``, double dot is almost always what you want.

editing an earlier commit
-------------------------

To amend a commit that isn't the latest, mark it ``edit`` in an interactive
rebase:

.. code-block:: console

        $ git rebase -i HEAD~3      # go back N commits

In the editor: ``pick`` keeps a commit as-is, ``edit`` stops so you can amend,
``squash`` merges into the previous commit, ``reword`` changes only the message.
When the rebase stops on an ``edit``:

.. code-block:: console

        $ # make your changes
        $ git add <files>
        $ git commit --amend -S --no-edit
        $ git rebase --continue

If you have unstaged work when starting, ``git stash`` first and ``git stash
pop`` after the rebase stops.

force-push safety
-----------------

Rebasing rewrites commit hashes, so updating the remote needs a force push.
Always prefer the safe variant:

.. code-block:: console

        $ git push --force-with-lease   # refuses if someone else pushed since your last fetch
        $ git push --force              # overwrites unconditionally - avoid on shared branches
