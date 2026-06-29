Python exercises
================

Practice tasks for the concepts in :doc:`python`. Each task lists the problem,
then a worked solution.

Lists
-----

Rotate a list ``k`` places to the right
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``rotate([1,2,3,4,5], 2)`` -> ``[4,5,1,2,3]``. Slicing with negative indices:
``nums[-k:]`` is the tail, ``nums[:-k]`` the rest.

.. code-block:: python

        def rotate(nums, k):
            k = k % len(nums)          # handles k > len
            return nums[-k:] + nums[:-k]

Remove duplicates preserving order
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``list(set(...))`` loses order; use a ``seen`` set for O(1) checks (or
``list(dict.fromkeys(items))``).

.. code-block:: python

        def dedup(items):
            seen, out = set(), []
            for x in items:
                if x not in seen:
                    seen.add(x); out.append(x)
            return out

Flatten a list of lists (one level)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The comprehension reads left-to-right like nested ``for`` loops.

.. code-block:: python

        def flatten(lists):
            return [item for sub in lists for item in sub]

Dicts
-----

Group by first letter
^^^^^^^^^^^^^^^^^^^^^^

``defaultdict(list)`` creates the empty list on first access.

.. code-block:: python

        from collections import defaultdict
        def group_by_letter(words):
            groups = defaultdict(list)
            for w in words:
                groups[w[0]].append(w)
            return dict(groups)

Invert a dict
^^^^^^^^^^^^^

.. code-block:: python

        def invert(d):
            return {v: k for k, v in d.items()}

Merge keeping the larger value
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dict keys behave like sets, so ``|`` gives the union; ``.get(k, 0)`` avoids
``KeyError``.

.. code-block:: python

        def merge_max(d1, d2):
            return {k: max(d1.get(k, 0), d2.get(k, 0)) for k in d1.keys() | d2.keys()}

Strings
-------

Palindrome (ignoring case and punctuation)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

        def is_palindrome(s):
            cleaned = "".join(c.lower() for c in s if c.isalnum())
            return cleaned == cleaned[::-1]

Most frequent character
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

        from collections import Counter
        def most_frequent(s):
            return Counter(s.replace(" ", "")).most_common(1)[0][0]

Functions
---------

A ``@timer`` decorator
^^^^^^^^^^^^^^^^^^^^^^

A decorator is a function that wraps another. ``@wraps`` preserves the original
name/docstring.

.. code-block:: python

        import time
        from functools import wraps
        def timer(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                start = time.time()
                result = func(*args, **kwargs)
                print(f"{func.__name__} took {time.time()-start:.2f}s")
                return result
            return wrapper

Call a function with only the kwargs it accepts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``inspect.signature`` introspection - the pattern behind dependency injection.

.. code-block:: python

        import inspect
        def call_with_applicable_args(func, **kwargs):
            params = inspect.signature(func).parameters
            return func(**{k: v for k, v in kwargs.items() if k in params})

Classes
-------

A simple stack
^^^^^^^^^^^^^^

.. code-block:: python

        class Stack:
            def __init__(self): self._items = []
            def push(self, x): self._items.append(x)
            def pop(self):
                if not self._items: raise IndexError("pop from empty stack")
                return self._items.pop()
            def peek(self): return self._items[-1]
            def is_empty(self): return not self._items
            def size(self): return len(self._items)

``__eq__`` and ``__hash__``
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Objects that are equal must hash equal. Define ``__hash__`` from the same fields
used in ``__eq__``; defining ``__eq__`` without ``__hash__`` makes the class
unhashable.

.. code-block:: python

        class Build:
            def __init__(self, stream, arch, version):
                self.stream, self.arch, self.version = stream, arch, version
            def __eq__(self, o):
                if not isinstance(o, Build): return NotImplemented
                return (self.stream, self.arch) == (o.stream, o.arch)
            def __hash__(self):
                return hash((self.stream, self.arch))

Error handling
--------------

Retry with exponential backoff
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

A decorator *factory* - ``retry(...)`` returns the decorator, which returns the
wrapper. Backoff doubles each attempt; the last attempt re-raises.

.. code-block:: python

        import time
        from functools import wraps
        def retry(max_attempts=3, base_delay=1):
            def decorator(func):
                @wraps(func)
                def wrapper(*args, **kwargs):
                    for attempt in range(1, max_attempts + 1):
                        try:
                            return func(*args, **kwargs)
                        except Exception:
                            if attempt == max_attempts: raise
                            time.sleep(base_delay * 2 ** (attempt - 1))
                return wrapper
            return decorator

Interview algorithms
--------------------

Two sum (hash map, O(n))
^^^^^^^^^^^^^^^^^^^^^^^^

For each number, ask "have I seen its complement?" - the dict answers in O(1).

.. code-block:: python

        def two_sum(nums, target):
            seen = {}
            for i, n in enumerate(nums):
                if target - n in seen: return [seen[target - n], i]
                seen[n] = i

Valid parentheses (stack)
^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: python

        def is_valid(s):
            stack, pairs = [], {")": "(", "]": "[", "}": "{"}
            for c in s:
                if c in "([{": stack.append(c)
                elif c in pairs:
                    if not stack or stack.pop() != pairs[c]: return False
            return not stack

Merge intervals (sort + linear scan)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

After sorting by start, overlapping intervals are adjacent.

.. code-block:: python

        def merge(intervals):
            intervals.sort(key=lambda x: x[0])
            merged = [intervals[0]]
            for start, end in intervals[1:]:
                if start <= merged[-1][1]:
                    merged[-1][1] = max(merged[-1][1], end)
                else:
                    merged.append([start, end])
            return merged

BFS shortest path in a grid
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

BFS guarantees the shortest path in an unweighted grid. Mark cells visited when
*adding* to the queue, not when popping.

.. code-block:: python

        from collections import deque
        def shortest_path(grid):
            if not grid or grid[0][0] == 1: return -1
            rows, cols = len(grid), len(grid[0])
            q, seen = deque([(0, 0, 1)]), {(0, 0)}
            while q:
                r, c, dist = q.popleft()
                if (r, c) == (rows - 1, cols - 1): return dist
                for dr, dc in [(0, 1), (0, -1), (1, 0), (-1, 0)]:
                    nr, nc = r + dr, c + dc
                    if 0 <= nr < rows and 0 <= nc < cols and grid[nr][nc] == 0 \
                            and (nr, nc) not in seen:
                        seen.add((nr, nc)); q.append((nr, nc, dist + 1))
            return -1

Binary search for a boundary (first >= target)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Record a candidate on a match and keep searching left. This is what
``bisect.bisect_left`` does internally.

.. code-block:: python

        def first_ge(nums, target):
            lo, hi, result = 0, len(nums) - 1, -1
            while lo <= hi:
                mid = (lo + hi) // 2
                if nums[mid] >= target:
                    result = mid; hi = mid - 1
                else:
                    lo = mid + 1
            return result

File I/O
--------

Top words in a file
^^^^^^^^^^^^^^^^^^^

.. code-block:: python

        import re
        from collections import Counter
        def top_words(path, n=5):
            text = open(path).read().lower()
            return Counter(re.findall(r'[a-z]+', text)).most_common(n)

Parse a ``key=value`` config
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``split("=", 1)`` splits on the first ``=`` only, so values may contain ``=``.

.. code-block:: python

        def parse_config(path):
            config = {}
            for line in open(path):
                line = line.strip()
                if not line or line.startswith("#"): continue
                key, value = line.split("=", 1)
                config[key.strip()] = value.strip()
            return config
