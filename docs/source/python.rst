Python essentials
=================

Data structures
---------------

.. code-block:: python

        nums = [1, 2, 3]
        nums.append(4); nums.insert(0, 0); nums.pop(); nums.pop(0)
        nums[-1]; nums[1:3]; nums[::-1]          # last / slice / reversed
        sorted(nums); nums.sort(reverse=True)    # new sorted list / in place
        3 in nums; nums.index(2); nums.count(1)

        # comprehensions
        [x**2 for x in range(10)]
        [x for x in range(20) if x % 2 == 0]
        [x for row in matrix for x in row]       # flatten
        {w: len(w) for w in words}               # dict comprehension
        {len(w) for w in words}                  # set comprehension

.. code-block:: python

        d = {"name": "adam"}
        d.get("age", 25)            # default instead of KeyError
        d.pop("name"); d.items()
        merged = d | {"team": "fcos"}            # merge (3.9+)

        from collections import defaultdict
        counts = defaultdict(int);  counts["a"] += 1     # missing key → 0
        groups = defaultdict(list); groups["t"].append("x")

        a = {1, 2, 3}; b = {2, 3, 4}
        a | b; a & b; a - b; a ^ b               # union/inter/diff/symdiff
        list(dict.fromkeys([3, 1, 2, 1]))        # dedup, preserve order → [3,1,2]

        x, y = (3, 4)                            # tuple unpacking
        from collections import namedtuple
        Point = namedtuple("Point", ["x", "y"]); Point(3, 4).x

Strings
-------

.. code-block:: python

        s = "hello world"
        s.strip(); s.split(","); ",".join(["a", "b"])
        s.replace("world", "py"); s.startswith("he"); s.find("world")

        name = "adam"
        f"hello {name}"        # use f-strings, not .format() or %
        f"{3.14159:.2f}"       # "3.14"
        f"{1000000:,}"         # "1,000,000"
        f"{'hi':>10}"          # right-align in width 10

Control flow
------------

.. code-block:: python

        x = "even" if n % 2 == 0 else "odd"          # ternary

        for item in items:                            # for/else
            if item == target: break
        else:
            print("not found")                        # runs if no break

        if (n := len(items)) > 10:                    # walrus (3.8+)
            print(f"too many: {n}")

        match command:                                # match/case (3.10+)
            case "quit":          exit()
            case "hello" | "hi":  greet()
            case _:               print("unknown")

Functions
---------

.. code-block:: python

        def greet(name, greeting="hello"):
            return f"{greeting} {name}"

        def log(*args, **kwargs):       # positional tuple + keyword dict
            ...
        add(*nums)                      # unpack list into args
        add(**opts)                     # unpack dict into kwargs

        double = lambda x: x * 2
        sorted(items, key=lambda x: x["name"])

        def process(items: list[str], count: int = 10) -> bool:   # type hints
            ...

Classes
-------

.. code-block:: python

        class Server:
            count = 0                        # class variable (shared)
            def __init__(self, host, port=8080):
                self.host = host; self.port = port
                Server.count += 1
            def url(self):  return f"http://{self.host}:{self.port}"
            def __repr__(self): return f"Server({self.host!r}, {self.port})"
            def __eq__(self, o): return (self.host, self.port) == (o.host, o.port)

        class SecureServer(Server):          # inheritance
            def __init__(self, host, port=443):
                super().__init__(host, port)
            def url(self): return f"https://{self.host}:{self.port}"

        from dataclasses import dataclass     # skip boilerplate
        @dataclass
        class Build:
            stream: str
            arch: str
            version: str = "0.0.1"            # auto __init__/__repr__/__eq__

Error handling
--------------

.. code-block:: python

        try:
            result = risky()
        except KeyError as e:
            print(f"missing key: {e}")
        except (TypeError, ValueError):
            print("bad type or value")
        except Exception as e:
            print(f"unexpected: {e}"); raise   # re-raise after logging
        else:
            print("no error")                  # runs if no exception
        finally:
            cleanup()

        class BuildError(Exception):           # custom exception
            pass

File I/O, YAML, JSON
--------------------

.. code-block:: python

        with open("data.txt") as f:
            content = f.read()
            lines = f.read().splitlines()      # without trailing \n
        with open("out.txt", "w") as f:        # "a" to append
            f.write("hello\n")

        import yaml
        config = yaml.safe_load(open("config.yaml"))

        import json
        data = json.loads('{"k": "v"}')        # str → dict
        text = json.dumps(data, indent=2)      # dict → str

Common algorithm patterns
-------------------------

.. code-block:: python

        # two pointers (sorted)
        def two_sum_sorted(nums, target):
            lo, hi = 0, len(nums) - 1
            while lo < hi:
                t = nums[lo] + nums[hi]
                if t == target: return [lo, hi]
                lo, hi = (lo + 1, hi) if t < target else (lo, hi - 1)

        # hash map for O(1) lookup
        def two_sum(nums, target):
            seen = {}
            for i, n in enumerate(nums):
                if target - n in seen: return [seen[target - n], i]
                seen[n] = i

        # sliding window
        def max_sum(nums, k):
            window = best = sum(nums[:k])
            for i in range(k, len(nums)):
                window += nums[i] - nums[i - k]
                best = max(best, window)
            return best

.. code-block:: python

        from collections import Counter, deque

        Counter("banana")                 # {'a':3,'n':2,'b':1}
        Counter(words).most_common(2)

        # stack - balanced parens
        def valid(s):
            stack, pairs = [], {")": "(", "]": "[", "}": "{"}
            for c in s:
                if c in "([{": stack.append(c)
                elif not stack or stack.pop() != pairs[c]: return False
            return not stack

        # BFS
        def bfs(graph, start):
            seen, q = {start}, deque([start])
            while q:
                node = q.popleft()
                for nb in graph[node]:
                    if nb not in seen: seen.add(nb); q.append(nb)
            return seen

        # binary search (or use bisect.bisect_left)
        def bsearch(nums, target):
            lo, hi = 0, len(nums) - 1
            while lo <= hi:
                mid = (lo + hi) // 2
                if nums[mid] == target: return mid
                lo, hi = (mid + 1, hi) if nums[mid] < target else (lo, mid - 1)
            return -1

Useful stdlib
-------------

.. code-block:: python

        for i, item in enumerate(items): ...        # index + value
        for a, b in zip(xs, ys): ...                # parallel iteration
        dict(zip(names, scores))                    # build dict from two lists
        any(x > 10 for x in nums); all(x > 0 for x in nums)
        max(words, key=len); min(people, key=lambda p: p["age"])

        from itertools import chain, combinations, permutations
        list(combinations([1, 2, 3], 2))            # [(1,2),(1,3),(2,3)]

        from functools import lru_cache
        @lru_cache(maxsize=None)
        def fib(n): return n if n < 2 else fib(n-1) + fib(n-2)

argparse
--------

Once a script grows past one or two positional arguments, switch from
``sys.argv`` to ``argparse`` - you get ``--help``, defaults and flags for free.

.. code-block:: python

        import argparse
        parser = argparse.ArgumentParser(description="Verify an RPM")
        parser.add_argument("version", help="OCP version (e.g. 4.19)")
        parser.add_argument("package", help="RPM name")
        parser.add_argument("--no-pull", action="store_true", help="Skip image pull")
        parser.add_argument("--rate-limit", type=float, default=2.0)
        parser.add_argument("issue_key", nargs="?", default=None)  # optional positional
        args = parser.parse_args()
        # args.version, args.no_pull (hyphens → underscores), ...

``action="store_true"`` makes a boolean switch (``True`` when the flag is given).

subprocess
----------

.. code-block:: python

        import subprocess
        r = subprocess.run(["ls", "-la"], capture_output=True, text=True)
        r.stdout; r.returncode
        subprocess.run(["make", "build"], check=True)   # raises on non-zero exit

pathlib
-------

.. code-block:: python

        from pathlib import Path
        p = Path("/home/user/project")
        p.exists(); p.is_dir()
        p / "data" / "file.md"          # join with /
        p.name; p.parent; p.suffix; p.stem
        list(p.glob("*.md")); list(p.rglob("*.py"))
        Path("file.txt").read_text(); Path("file.txt").write_text("hi\n")
        Path("out/sub").mkdir(parents=True, exist_ok=True)

Version-string parsing
----------------------

Use ``int()`` so comparisons are numeric, not alphabetical
(``"9" > "12"`` is ``True`` - wrong; ``9 > 12`` is ``False`` - right), and guard
against missing parts:

.. code-block:: python

        parts = version.split(".")               # "4.22.0-ec.5" → ['4','22','0-ec','5']
        major = int(parts[0]) if len(parts) >= 1 else 0
        minor = int(parts[1]) if len(parts) >= 2 else 0

Tuple comparison compares element by element - ideal for versions:
``(4, 22) > (4, 12)`` is ``True``; ``(4, 22) > (5, 1)`` is ``False``. When
mapping OCP → RHEL, check ``major >= 5`` separately so a future major version is
handled, rather than relying on ``minor >= 22`` alone.

Complexity quick reference
--------------------------

.. list-table::
   :header-rows: 1

   * - Operation
     - list
     - dict/set
   * - index / key lookup
     - O(1)
     - O(1)
   * - search (``in``)
     - O(n)
     - O(1)  ← why you use sets
   * - append / add
     - O(1)
     - O(1)
   * - insert / pop at front
     - O(n)
     - - (use ``deque`` for O(1))
   * - sort
     - O(n log n)
     - -
