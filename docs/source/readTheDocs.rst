How this site is built
======================

This site is written in reStructuredText, built with **Sphinx**, and published
automatically by **ReadTheDocs** on every push to ``main``.

- ReadTheDocs Community: https://readthedocs.org/
- Sphinx reStructuredText primer:
  https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html

Project layout
--------------

- ``docs/source/*.rst`` - one file per topic
- ``docs/source/index.rst`` - the landing page and the ``toctree``
- ``docs/source/conf.py`` - Sphinx config (theme, extensions)
- ``.readthedocs.yaml`` - the ReadTheDocs build configuration
- ``docs/requirements.txt`` - Python deps installed during the build

Adding a page
-------------

Drop a ``.rst`` file in ``docs/source/`` and add its name (without the
extension) to the ``toctree`` in ``index.rst``. Build locally to check it
renders:

.. code-block:: console

   $ pip install -r docs/requirements.txt
   $ sphinx-build -b html docs/source docs/_build
   $ # open docs/_build/index.html

Enabling an extension
---------------------

Add it to the ``extensions`` list in ``conf.py`` and to
``docs/requirements.txt`` so ReadTheDocs installs it on its build.
