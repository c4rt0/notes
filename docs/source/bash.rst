Bash — variables, arrays & eval
===============================

Notes on the parts of bash that trip people up: word splitting, why ``eval``
exists, and why arrays are almost always the better answer.

How the shell processes a command
----------------------------------

Before running anything, bash goes through these steps **in order**:

.. code-block:: console

        1. Parse the line
        2. Expand variables ($VAR becomes its value)
        3. Word split (break into arguments by whitespace)
        4. Quote removal (strip quotes used for grouping)
        5. Execute the command with the resulting arguments

This order is why strings, arrays and ``eval`` behave the way they do.

Variables (strings)
-------------------

.. code-block:: bash

        NAME="hello world"
        echo $NAME      # echo gets 2 args: "hello" and "world"
        echo "$NAME"    # echo gets 1 arg: "hello world"

**Rule: always double-quote your variables** (``"$VAR"``) unless you
specifically want word splitting.

.. code-block:: bash

        MY_VAR=$(whoami)           # command substitution — captures output
        unset MY_VAR               # removes the variable entirely

        echo "${MY_VAR}_suffix"    # braces stop the var name running into the suffix
        echo "${#MY_VAR}"          # length
        echo "${MY_VAR^^}"         # uppercase     (,, = lowercase)
        echo "${MY_VAR/hello/hi}"  # replace first match
        echo "${MY_VAR:0:5}"       # substring (offset:length)

The word splitting problem
--------------------------

.. code-block:: bash

        CMD="ls -la /tmp"
        $CMD                       # works — bash splits into: ls, -la, /tmp

        CMD="grep 'hello world' file.txt"
        $CMD
        # bash splits into: grep, 'hello, world', file.txt
        # the quotes inside the string are NOT processed — they're just characters

**Why?** Variable expansion (step 2) happens *before* quote removal (step 4).
By the time bash sees the quotes inside ``$CMD`` they are already plain
characters, not grouping markers.

eval — the nuclear option
-------------------------

``eval`` takes a string and runs it through the **entire** parsing pipeline
again, from step 1:

.. code-block:: bash

        CMD="grep 'hello world' file.txt"
        $CMD         # WRONG: grep sees 'hello as the pattern
        eval "$CMD"  # RIGHT: eval re-parses, quotes are honoured

It executes **anything** in the string, which is exactly why it's dangerous:

.. code-block:: bash

        user_input="hello; rm -rf /"
        eval "echo $user_input"    # runs BOTH commands

**Rule: never eval untrusted input.** Reserve it for the rare cases of dynamic
variable names or re-parsing quotes you control.

Arrays — the proper solution
----------------------------

Arrays keep each element as a separate word, so there is no word-splitting
issue to work around.

.. code-block:: bash

        FRUITS=(apple banana cherry)   # create
        EMPTY=()                       # empty
        COLORS+=(red)                  # append

        echo "${FRUITS[0]}"            # single element (0-indexed)
        echo "${FRUITS[@]}"            # ALL elements, each a separate word
        echo "${#FRUITS[@]}"           # number of elements
        echo "${!FRUITS[@]}"           # all indices

``[@]`` vs ``[*]`` — the critical difference
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

        ITEMS=("hello world" "foo bar")

        for x in "${ITEMS[@]}"; do echo "[$x]"; done   # [hello world] / [foo bar]
        for x in "${ITEMS[*]}"; do echo "[$x]"; done   # [hello world foo bar]  (one string)
        for x in ${ITEMS[@]};   do echo "[$x]"; done   # [hello]/[world]/[foo]/[bar]  (split!)

**Rule: always use** ``"${ARRAY[@]}"`` **with double quotes and** ``@``.

Arrays as command arguments
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This is the key use case — building a command with optional arguments:

.. code-block:: bash

        # The eval way (avoid)
        EXTRA_ARGS=""
        [ "$verbose" = "true" ] && EXTRA_ARGS="--debug -vvvv"
        eval "tmt run $EXTRA_ARGS --all"

        # The array way (correct)
        EXTRA_ARGS=()
        [ "$verbose" = "true" ] && EXTRA_ARGS=(--debug -vvvv)
        tmt run "${EXTRA_ARGS[@]}" --all
        # empty: tmt run --all      set: tmt run --debug -vvvv --all

``"${EXTRA_ARGS[@]}"`` on an empty array expands to *nothing* — not an empty
string — so there's no stray gap, and it's safe under ``set -u``.

Quick reference
---------------

.. list-table::
   :header-rows: 1

   * - What
     - Syntax
   * - Create array
     - ``ARR=(a b c)``
   * - Empty array
     - ``ARR=()``
   * - Read element
     - ``${ARR[0]}``
   * - All elements
     - ``"${ARR[@]}"``
   * - Length
     - ``${#ARR[@]}``
   * - Append
     - ``ARR+=(x)``
   * - Indices
     - ``${!ARR[@]}``

When to use what
----------------

.. list-table::
   :header-rows: 1

   * - Scenario
     - Use
   * - Simple value
     - ``VAR="value"``
   * - Optional command arguments
     - ``ARR=()`` + ``"${ARR[@]}"``
   * - Building commands from strings
     - **avoid** — use arrays
   * - Dynamic variable names
     - ``eval`` (rare, be careful)
   * - Must re-parse quotes in a string
     - ``eval`` (last resort)
