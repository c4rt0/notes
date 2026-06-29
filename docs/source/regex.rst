Regular expressions - Python ``re`` + bash ``grep``/``sed``
============================================================

Each section shows the concept in Python first, then the bash equivalent.

Literal matching & metacharacters
----------------------------------

Most characters match themselves. A few are **metacharacters** and need
escaping with ``\`` to match literally: ``. ^ $ \ * + ? [ ] ( ) { } |``.

.. code-block:: python

        import re
        re.search(r'core', 'fedora-coreos-config')   # <Match 'core'>  (anywhere)
        re.match(r'core', 'fedora-coreos-config')     # None (must match at start)
        re.fullmatch(r'core', 'core')                 # <Match 'core'> (whole string)

**Rule of thumb:** use ``re.search()`` unless you need start-of-string or
full-string matching. Always use **raw strings** (``r"..."``) for patterns so
backslash escapes like ``\b`` aren't processed by Python first. ``re.escape(s)``
escapes every metacharacter in a string.

In bash, ``grep`` has flavors:

.. list-table::
   :header-rows: 1

   * - Flag
     - Flavor
     - Use when
   * - (none)
     - BRE
     - simple literal matching
   * - ``-E``
     - ERE
     - ``+ ? {} ()`` without escaping - the daily driver
   * - ``-P``
     - PCRE
     - need ``\d``, ``\w``, lookaround - closest to Python
   * - ``-F``
     - fixed
     - literal strings containing metacharacters

Character classes
-----------------

``[...]`` matches **one** character from the set.

.. code-block:: python

        re.findall(r'[aeiou]', 'coreos')        # ['o', 'e', 'o']
        re.findall(r'[0-9]', 'rpm-4.19')        # ['4', '1', '9']
        # ranges & negation: [a-z] [A-Z] [0-9] [a-zA-Z0-9] [^0-9]

Inside ``[...]``: ``-`` is a range (literal if first/last), ``^`` negates only
at the start, ``]`` is literal if first.

.. list-table::
   :header-rows: 1

   * - Shorthand
     - Equivalent
     - Meaning
   * - ``\d`` / ``\D``
     - ``[0-9]`` / ``[^0-9]``
     - digit / not digit
   * - ``\w`` / ``\W``
     - ``[a-zA-Z0-9_]`` / negated
     - word char / not
   * - ``\s`` / ``\S``
     - ``[ \t\n\r\f\v]`` / negated
     - whitespace / not

ERE (``grep -E``) has no ``\d``/``\w`` - use POSIX classes (``[[:digit:]]``,
``[[:alpha:]]``, ``[[:alnum:]]``, ``[[:space:]]``) or switch to ``grep -P``.

Quantifiers
-----------

.. list-table::
   :header-rows: 1

   * - Quantifier
     - Meaning
   * - ``*`` / ``+`` / ``?``
     - 0+ / 1+ / 0 or 1
   * - ``{n}`` / ``{n,}`` / ``{n,m}``
     - exactly n / n or more / between n and m

By default quantifiers are **greedy** (match as much as possible); add ``?`` to
make them **lazy**:

.. code-block:: python

        text = '<b>bold</b> and <i>italic</i>'
        re.findall(r'<.*>',  text)   # ['<b>bold</b> and <i>italic</i>']  (greedy)
        re.findall(r'<.*?>', text)   # ['<b>', '</b>', '<i>', '</i>']     (lazy)
        re.findall(r'"([^"]*)"', 'a "x" b "y"')   # ['x', 'y']  (often cleaner & faster)

In bash, ``-o`` prints only the match; ERE has no lazy quantifiers (use ``-P``):

.. code-block:: bash

        echo "rpm-4.19-3.el9" | grep -oE '[0-9]+\.[0-9]+'   # 4.19
        echo '<b>x</b>'       | grep -oP '<.*?>'            # <b>  </b>

Groups and capturing
--------------------

Parentheses group elements and **capture** the matched text.

.. code-block:: python

        m = re.search(r'(?P<major>\d+)\.(?P<minor>\d+)', 'rhel-9.8')
        m.group(0)       # '9.8'   (entire match)
        m.group('major') # '9'
        m.groupdict()    # {'major': '9', 'minor': '8'}

        # findall returns group contents when groups exist:
        re.findall(r'(\d+)\.(\d+)', 'rhel-9.8 rhel-10.2')   # [('9','8'), ('10','2')]
        re.findall(r'(?:rhel|fedora)-\d+', 'rhel-9 fedora-42')  # full match (non-capturing)
        re.search(r'<(\w+)>.*?</\1>', '<b>bold</b>')        # backreference \1

In bash, ``sed -E`` captures with ``()`` and references with ``\1``, ``\2``:

.. code-block:: bash

        echo "ignition-2.20.0-3.el9" | sed -E 's/.*-([0-9.]+)-.*/\1/'   # 2.20.0
        echo "Mabe, Dusty"           | sed -E 's/([^,]+), (.*)/\2 \1/'   # Dusty Mabe

Alternation and anchors
-----------------------

``|`` means "or". Anchors match a **position**, not a character: ``^`` (start),
``$`` (end), ``\b`` (word boundary), ``\B`` (not a boundary).

.. code-block:: python

        re.search(r'^rhel', 'rhel-9.8')             # match
        re.search(r'\.conf$', 'build-9.8.conf')     # match
        re.findall(r'\bcore\b', 'coreos core util') # ['core']  (whole word only)

For repeated use, compile once with ``re.compile(r'...')`` - it returns a
pattern object with the same methods and is faster in loops. In bash, ``grep -w``
matches whole words.

Substitution
------------

.. code-block:: python

        re.sub(r'\d+', 'X', 'rhel-9.8')                  # 'rhel-X.X'
        re.sub(r'(\d+)\.(\d+)', r'\2.\1', 'rhel-9.8')    # 'rhel-8.9'  (swap groups)
        re.sub(r'\d+', 'X', '1.2.3', count=1)            # 'X.2.3'
        re.sub(r'\d+', lambda m: str(int(m.group())*2), 'v4')  # function replacement
        re.split(r'[-.]', 'ignition-2.20.0')             # ['ignition','2','20','0']

.. code-block:: bash

        echo "rhel-9.8" | sed -E 's/[0-9]+/X/g'                 # rhel-X.X (all)
        echo "rhel-9.8" | sed -E 's/([0-9]+)\.([0-9]+)/\2.\1/'  # rhel-8.9
        sed -i -E 's/old/new/g' file.txt                        # edit in place

Lookahead and lookbehind
------------------------

Zero-width assertions - they check around a position without consuming it.

.. list-table::
   :header-rows: 1

   * - Syntax
     - Meaning
   * - ``(?=...)`` / ``(?!...)``
     - positive / negative lookahead (what follows)
   * - ``(?<=...)`` / ``(?<!...)``
     - positive / negative lookbehind (what precedes)

.. code-block:: python

        re.findall(r'\d+\.\d+(?=-el)', 'a-4.19-el9 b-3.2-fc42')  # ['4.19']
        re.findall(r'(?<=el)\d+', 'rhel-9.8 el10')               # ['9', '10']
        re.search(r'(?<=STREAM=)\w+', 'STREAM=stable').group()   # 'stable'

In Python, lookbehind must be **fixed-width** (no ``*``/``+``/``{n,m}`` inside).
ERE has no lookaround; use ``grep -P``.

Multiline and flags
-------------------

.. code-block:: python

        re.findall(r'rhel', 'RHEL rhel', re.IGNORECASE)        # ['RHEL', 'rhel']
        re.findall(r'^VERSION_ID=(.+)$', text, re.MULTILINE)   # ^/$ at line bounds
        re.search(r'<tag>(.+)</tag>', text, re.DOTALL)         # . also matches \n
        re.findall(r'^name=(.+)$', text, re.I | re.M)          # combine with |
        re.findall(r'(?i)rhel', 'RHEL')                        # inline flag

bash: ``grep -i`` is case-insensitive, and ``grep`` is line-oriented so ``^``/``$``
work per line already.

Common patterns
---------------

.. code-block:: python

        # RPM NVR (name can contain hyphens; version starts with a digit)
        NVR = r'^(.+)-(\d[\d.]*\d)-(\d+\.\w+)$'
        re.match(NVR, 'ignition-2.20.0-3.el9').groups()  # ('ignition','2.20.0','3.el9')

        # version string (optional patch)
        re.search(r'(\d+)\.(\d+)(?:\.(\d+))?', 'OCP 4.22.0').groups()  # ('4','22','0')

        # IP address (doesn't validate 0-255)
        re.findall(r'\b\d{1,3}(?:\.\d{1,3}){3}\b', 'host 192.168.1.1')  # ['192.168.1.1']

        # /etc/os-release line (handles quoted & unquoted values)
        re.findall(r'^(\w+)=("?)(.+?)\2$', text, re.M)

Quick reference
---------------

.. list-table::
   :header-rows: 1

   * - ``re`` function
     - Use
   * - ``re.search`` / ``re.match`` / ``re.fullmatch``
     - find first / at start / whole string
   * - ``re.findall`` / ``re.finditer``
     - all matches (list / iterator of Match)
   * - ``re.sub`` / ``re.split``
     - replace / split on pattern
   * - ``re.compile`` / ``re.escape``
     - compile for reuse / escape metacharacters

Match object: ``m.group(n)``, ``m.groups()``, ``m.groupdict()``,
``m.start()``/``m.end()``, ``m.span()``.

.. list-table::
   :header-rows: 1

   * - grep flag
     - Meaning
   * - ``-E`` / ``-P`` / ``-F``
     - ERE / PCRE / fixed string
   * - ``-i`` / ``-o`` / ``-v`` / ``-w``
     - ignore case / only match / invert / whole word
   * - ``-c`` / ``-l`` / ``-n`` / ``-r`` / ``-q``
     - count / list files / line numbers / recursive / quiet

sed: ``-E`` (extended), ``-i`` (in place), ``g`` (all occurrences in ``s///g``),
``I`` (case-insensitive).
