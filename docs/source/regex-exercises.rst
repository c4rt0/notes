Regex exercises
===============

Practice tasks for :doc:`regex`. Each has a Python solution and, where useful,
a bash equivalent.

Character classes & anchors
---------------------------

Filter RPMs by prefix
^^^^^^^^^^^^^^^^^^^^^^

For a literal prefix, ``str.startswith()`` is cleaner; use regex when the prefix
itself is a pattern.

.. code-block:: python

        import re
        def find_rpms(rpms, prefix):
            pat = re.compile(r'^' + re.escape(prefix))
            return [r for r in rpms if pat.match(r)]

Extract individual digits
^^^^^^^^^^^^^^^^^^^^^^^^^^

``\d`` (single) vs ``\d+`` (groups of digits):

.. code-block:: python

        def extract_digits(s):
            return [int(d) for d in re.findall(r'\d', s)]   # "4.19" -> [4,1,9]

Quantifiers
-----------

Extract version numbers (X.Y or X.Y.Z)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Use a **non-capturing** group so ``findall`` returns the full match, not the
optional ``.Z`` group:

.. code-block:: python

        def extract_versions(text):
            return re.findall(r'\d+\.\d+(?:\.\d+)?', text)

Extract quoted strings (greedy vs lazy)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``[^"]*`` (any char except quote) is correct and faster than lazy ``.*?``:

.. code-block:: python

        def extract_quoted(s):
            return re.findall(r'"([^"]*)"', s)

Groups and capturing
--------------------

Parse an RPM NVR
^^^^^^^^^^^^^^^^

Greedy ``(.+)`` for the name backtracks to the last hyphen before a digit, so it
handles hyphenated names; ``\d[\d.]*\d`` keeps the version digit-bounded.

.. code-block:: python

        def parse_nvr(nvr):
            m = re.match(r'(?P<name>.+)-(?P<version>\d[\d.]*\d)-(?P<release>\d+\.\w+)$', nvr)
            if not m:
                raise ValueError(f"Invalid NVR: {nvr}")
            return m.groupdict()

Parse os-release (quoted and unquoted)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``("?)`` captures an optional opening quote; ``\2`` backreferences the same
character at the end:

.. code-block:: python

        def parse_os_release(text):
            pairs = re.findall(r'^(\w+)=("?)(.+?)\2$', text, re.M)
            return {k: v for k, _, v in pairs}

Alternation and anchors
-----------------------

Filter by OS variant
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

        def filter_images(images):
            return [i for i in images if re.search(r'rhel|fedora', i)]

Whole-word match
^^^^^^^^^^^^^^^^

``\bcore\b`` matches ``core`` and ``core-utils`` but not ``coreos``/``coreutils``
(``-`` is a word boundary, ``o`` is not). In bash, ``grep -w`` does the same.

.. code-block:: python

        def find_core(packages):
            return [p for p in packages if re.search(r'\bcore\b', p)]

Substitution
------------

Mask IP addresses
^^^^^^^^^^^^^^^^^

.. code-block:: python

        def mask_ips(log):
            return re.sub(r'\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b', '[REDACTED]', log)

Strip the minor version from repo names
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Capture the part to keep, match-but-don't-capture the rest:

.. code-block:: python

        def rewrite_repos(repos):
            return [re.sub(r'(rhel-\d+)\.\d+', r'\1', r) for r in repos]
        # "rhel-9.8-baseos" -> "rhel-9-baseos"; "epel-9" unchanged

Lookahead and lookbehind
------------------------

Extract a value after a label
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Lookbehind asserts ``KEY=`` precedes the match without including it; ``\b`` keeps
``ARCH`` from matching inside ``BASEARCH``. The capture-group version is more
common in practice:

.. code-block:: python

        def get_value(meta, key):
            m = re.search(rf'\b{re.escape(key)}=(\S+)', meta)
            return m.group(1) if m else None

Match a version only before a suffix
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

        def el9_versions(artifacts):
            text = '\n'.join(artifacts)
            return re.findall(r'\d+\.\d+\.\d+(?=-\d+\.el9)', text)

Multiline and flags
-------------------

Parse ARG definitions from a Containerfile
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``re.M`` makes ``^`` match each line; the ``("?)...\2`` trick handles optional
quotes (empty value matches too):

.. code-block:: python

        def extract_args(containerfile):
            pairs = re.findall(r'^ARG\s+(\w+)=("?)([^"\n]*)\2', containerfile, re.M)
            return {k: v for k, _, v in pairs}

Real-world combined
-------------------

Parse kola test results
^^^^^^^^^^^^^^^^^^^^^^^^

``finditer`` yields Match objects, so named groups are available; convert types
as you build the dict:

.. code-block:: python

        def parse_kola_results(output):
            pat = re.compile(r'--- (?P<status>PASS|FAIL|SKIP): (?P<name>\S+) \((?P<duration>[\d.]+)s\)')
            return [{"name": m["name"], "status": m["status"],
                     "duration": float(m["duration"])} for m in pat.finditer(output)]

Validate and normalize image references
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``fullmatch`` rejects malformed strings; the optional tag group lets you add a
default:

.. code-block:: python

        IMAGE_RE = re.compile(
            r'(?P<registry>[\w.-]+(?::\d+)?)'   # registry (optional port)
            r'/(?P<path>[\w./-]+)'               # image path
            r'(?::(?P<tag>[\w.+-]+))?'           # optional tag
        )
        def normalize_images(images):
            out = []
            for img in images:
                m = IMAGE_RE.fullmatch(img)
                if m:
                    out.append(img if m["tag"] else f"{img}:latest")
            return out

Extract failed tests from a CI log
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Match the error line followed by the FAIL line (relies on consistent Go test
output):

.. code-block:: python

        def extract_failures(ci_log):
            return [{"test": m.group(2), "error": m.group(1)}
                    for m in re.finditer(r'Error: (.+)\n--- FAIL: (\S+)', ci_log)]
