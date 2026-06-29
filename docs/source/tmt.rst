TMT (Test Management Tool)
==========================

What TMT is and why it exists
-----------------------------

TMT is a test framework used to run tests on RPM packages. When a package
is built, the CI pipeline looks for TMT tests and runs them as **gating**
tests - if they pass, the RPM can enter a compose; if they fail, it's blocked.

.. code-block:: text

        Developer pushes     build the RPM    CI runs          Package enters
        code to dist-git --> (Koji/Brew)  --> gating tests  --> a compose
                                              (TMT)

TMT uses **FMF** (Flexible Metadata Format) - YAML files with a ``.fmf``
extension. Think of FMF as the language and TMT as the runner.

.. code-block:: bash

        dnf install tmt           # preferred
        pip install tmt           # alternative
        dnf install tmt+all       # all provisioning methods

The three building blocks
-------------------------

``.fmf/version`` - the root marker
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A file containing just ``1``. Like ``.git/`` marks a git repo,
``.fmf/version`` marks an FMF tree. Without it, TMT finds nothing.

.. code-block:: bash

        tmt init              # creates .fmf/version automatically
        # or manually:
        mkdir .fmf && echo 1 > .fmf/version

Tests - what to run
^^^^^^^^^^^^^^^^^^^

A ``.fmf`` file with a ``test:`` key - the actual script that gets executed.

.. code-block:: yaml

        # tests/tmt/tests/core/core.fmf
        summary: check the package is installed
        tag:
          - smoke
        test: |
          set -xeuo pipefail
          rpm -q console-login-helper-messages

- ``summary:`` - human-readable description (must be under 50 chars; lint C001)
- ``tag:`` - labels; ``smoke`` is the convention for gating tests
- ``test:`` - shell script. Exit 0 = pass, non-zero = fail

TMT scans the whole tree under ``.fmf/version``; the path is convention only.

Plans - how to find and run tests
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: yaml

        # tests/tmt/plans/smoke.fmf
        summary: Basic smoke test
        tag:
          - smoke
        discover:
          how: fmf
          filter: "tag: smoke"
        execute:
          how: tmt

When multiple plans live in the same directory, TMT combines their phases -
a ``main.fmf`` with a ``prepare:`` runs before ``smoke.fmf``'s
``discover:`` + ``execute:``.

Plan phases
-----------

Plans have six phases. You typically write the first four; ``report``,
``finish`` and ``cleanup`` are handled automatically.

- ``prepare:`` - set up the environment. ``how: install`` lets TMT handle the
  package manager; ``how: shell`` runs arbitrary commands.
- ``discover:`` - find the tests. ``how: fmf`` scans from the FMF root;
  ``url:`` clones a remote repo and scans there (used in dist-git plans);
  ``filter:`` limits which tests run.
- ``execute:`` - run the tests. Almost always ``how: tmt``.
- ``report:`` / ``finish:`` / ``cleanup:`` - automatic.

.. code-block:: yaml

        prepare:
          - name: Install the package
            how: install
            package: my-package
          - name: Run a setup script
            how: shell
            script: |
              echo "SOME_VAR=value" > /tmp/config

Running TMT locally
-------------------

.. code-block:: bash

        tmt tests ls       # list discovered tests
        tmt plans ls       # list discovered plans
        tmt lint           # check for errors/warnings

        # Run all plans in a container
        tmt run --all provision --how container

        # Verbose (recommended - see below)
        tmt run --all --debug -vvvv provision --how container

        # Only smoke-tagged plans
        tmt run --all plan --filter 'tag: smoke' provision --how container

        # Specific image
        tmt run --all provision --how container --image fedora:latest

        # Directly on your machine (careful!)
        tmt run --all provision --how local --feeling-safe

``provision --how container`` spins up a podman container, runs everything
inside it, then tears it down.

Verbosity - ``--debug -vvvv``
-----------------------------

**Always use** ``--debug -vvvv`` **in CI workflows.** Without them, TMT output
is minimal and you can't tell if the test actually ran - a reviewer looking at
CI logs would have no evidence of execution.

- ``-vvvv`` adds user-facing detail: plan discovery, test selection,
  provisioning info, actual test output, pass/fail results, final summary.
- ``--debug`` adds developer detail: workdir creation, queue management,
  connection attempts, internal queries.

.. code-block:: text

        Found 1 plan.

        /tests/tmt/plans/smoke
            discover
                summary: 1 test selected
            provision
                how: container
                distro: Fedora Linux 44 (Container Image)
            prepare
                cmd: dnf5 install -y my-package
            execute
                00:00:00 pass /tests/tmt/tests/core/core (on default-0) [1/1]
            report
                summary: 1 test passed

        total: 1 test passed

GitHub Actions workflow for TMT
-------------------------------

Triggers TMT in a container on the GitHub Actions runner (not Testing Farm):

.. code-block:: yaml

        name: TMT Tests
        on:
          push:
            branches: [main]
          pull_request:
            branches: [main]
          workflow_dispatch:
            inputs:
              plan_filter:
                description: "Test plan filter, e.g. tag:smoke (empty = run all)"
                required: false
                default: ''
        permissions:
          contents: read
        jobs:
          tmt-tests:
            runs-on: ubuntu-latest
            steps:
              - uses: actions/checkout@v6
              - name: Set additional paths
                run: echo "$HOME/.local/bin" >> $GITHUB_PATH
              - name: Install dependencies
                run: |
                  set -xeuo pipefail
                  sudo apt-get update && sudo apt-get install -y podman
                  pip install --user "tmt[provision]"
              - name: Run TMT tests
                run: |
                  set -xeuo pipefail
                  PLAN_FILTER=()
                  if [ -n "${{ github.event.inputs.plan_filter }}" ]; then
                    PLAN_FILTER=(plan --filter '${{ github.event.inputs.plan_filter }}')
                  fi
                  tmt run --all --debug -vvvv provision --how container "${PLAN_FILTER[@]}"

Key details:

- **$GITHUB_PATH** - ``pip install --user`` puts ``tmt`` in ``~/.local/bin``,
  not on ``$PATH`` by default; the "Set additional paths" step adds it.
- **set -xeuo pipefail** - ``-x`` prints commands, ``-e`` exits on error,
  ``-u`` errors on unset variables, ``-o pipefail`` fails if any pipe command
  fails.
- **PLAN_FILTER** uses a bash *array*, not a string with ``eval`` - see the
  :doc:`bash` page. An empty array expands to nothing; a populated one expands
  to ``plan --filter 'tag: smoke'``.

GitHub Actions vs Testing Farm vs Konflux
-----------------------------------------

.. list-table::
   :header-rows: 1

   * - System
     - Where it runs
     - What it tests
     - Who configures it
   * - GitHub Actions
     - GH runner
     - Source code
     - Developer (workflow YAML)
   * - Testing Farm
     - Shared test infra
     - Built RPMs
     - CI auto-discovers
   * - Konflux
     - Tekton clusters
     - Container images
     - Team (Tekton + UI)

The two-layer pattern - upstream + dist-git
-------------------------------------------

Gating tests are split across two repos. The **upstream** repo holds the tests
and plans; the **dist-git** repo holds plans that *discover from upstream* via
``discover.url``, so no test code is duplicated:

.. code-block:: yaml

        discover:
          - name: Upstream tests
            how: fmf
            url: https://github.com/coreos/console-login-helper-messages.git
            filter: 'tag: smoke'

``gating.yaml`` - skip it
-------------------------

CI auto-detects TMT tests in dist-git repos, so ``gating.yaml`` is being phased
out. Don't add it for new packages.

Useful commands
---------------

.. code-block:: bash

        tmt init                                # create .fmf/version
        tmt tests ls                            # list discovered tests
        tmt plans ls                            # list discovered plans
        tmt lint                                # validate structure
        tmt run --all --debug -vvvv provision --how container
        tmt run discover                        # dry run: just discover
        tmt test create --template shell /tests/smoke   # scaffold a test
        tmt plan create --template mini /plans/smoke    # scaffold a plan

Docs
----

- TMT: https://tmt.readthedocs.io/
- FMF: https://fmf.readthedocs.io/
- Plan spec (phases): https://tmt.readthedocs.io/en/stable/spec/plans.html
