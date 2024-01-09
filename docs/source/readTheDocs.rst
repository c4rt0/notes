HOW TO: Read the Docs 
=====================

1. Sign up to Read the Docs Community:
https://readthedocs.org/

2. Read the Docs Doccumentation:
https://docs.readthedocs.io/en/stable/tutorial/index.html


Usage
-------

.. _installation:

Installation
------------

To use Lumache, first install it using pip:

.. code-block:: console

   (.venv) $ pip install lumache

Creating recipes
----------------

To retrieve a list of random ingredients,
you can use the ``lumache.get_random_ingredients()`` function:

.. autofunction:: lumache.get_random_ingredients

The ``kind`` parameter should be either ``"meat"``, ``"fish"``,
or ``"veggies"``. Otherwise, :py:func:`lumache.get_random_ingredients`
will raise an exception.

.. autoexception:: lumache.InvalidKindError

For example:

>>> import lumache
>>> lumache.get_random_ingredients()
['shells', 'gorgonzola', 'parsley']


Adding extensions
---------------

`TODO <https://github.com/c4rt0/notes/blob/main/docs/source/conf.py#L20-L23>`_

Adding images
---------------

`Example <https://github.com/readthedocs/readthedocs.org/blob/9e2e78653c5a7a79d1ae41cf016de7516a7d30d0/docs/user/tutorial/index.rst?plain=1#L60-L63>`_