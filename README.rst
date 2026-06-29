Linux Developer's Notes
=======================

Practical, condensed notes on the tools I use as a software and
operating-systems engineer - heavy on Fedora, CoreOS/RHCOS, containers and CI.
Each page is the distilled version of something I had to work out once and did
not want to re-learn from scratch.

**Live site:** https://apiaseck.readthedocs.io/

Topics
------

- **Shell & scripting** - bash (variables, arrays, eval), forgotten bash
  commands, zsh/bash config, regular expressions (Python ``re`` + grep/sed)
- **Containers & CoreOS** - podman & skopeo, COSA / building RHCOS, Fedora
  Silverblue
- **CI & build tooling** - TMT, Jenkins, Groovy & Job DSL, YAML config-as-code
- **Languages** - Python essentials, Java (for reading Jenkins plugin source)
- **Workstation** - git workflows, GPG & SSH keys
- **Other** - RHCSA notes, training your own models

Building locally
----------------

The site is built with `Sphinx <https://www.sphinx-doc.org/>`_ from
reStructuredText and published automatically by
`ReadTheDocs <https://readthedocs.io>`_ on every push to ``main``.

.. code-block:: console

    $ python -m venv .venv && source .venv/bin/activate
    $ pip install -r docs/requirements.txt
    $ sphinx-build -b html docs/source docs/_build
    $ # then open docs/_build/index.html

Adding a page
-------------

Drop a ``.rst`` file into ``docs/source/`` and add its name to the ``toctree``
in ``docs/source/index.rst``. Convert Markdown to reStructuredText as you go:
``#`` headings become underlined titles, fenced code blocks become
``.. code-block::`` directives, and ``[text](url)`` links become
```` `text <url>`_ ````.
