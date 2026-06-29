junitparser - JUnit XML in Python
=================================

``junitparser`` creates and reads JUnit XML files - the standard format CI
systems (Jenkins, GitHub Actions, …) use to display which tests passed, failed
or were skipped.

- Docs: https://junitparser.readthedocs.io/
- PyPI: https://pypi.org/project/junitparser/

Install with ``pip install junitparser``.

The model
---------

The structure is **document > suites > test cases**:

- ``TestCase`` - one test (e.g. ``test_login``)
- ``TestSuite`` - a group of test cases
- ``JUnitXml`` - the whole document (can hold multiple suites)

.. code-block:: python

        from junitparser import TestCase, TestSuite, JUnitXml

        tc = TestCase("my-test")
        suite = TestSuite("my-suite")
        suite.add_testcase(tc)

        xml = JUnitXml()
        xml.add_testsuite(suite)
        xml.write("results.xml")        # or "/dev/stdout" to print

Test case details
-----------------

.. code-block:: python

        tc = TestCase("test_login", "auth.tests", 0.5)   # name, classname, time(s)
        tc.system_out = "Login successful"               # stdout shown in Jenkins
        tc.system_err = "DEBUG: connecting..."           # stderr

Pass / fail / skip
------------------

A test with no result is a pass. To mark failure or skip, set ``.result`` to a
list (a list because a test can have multiple results); use ``.text`` for the
detailed output:

.. code-block:: python

        from junitparser import Failure, Skipped

        fail = Failure("Network unreachable")
        fail.text = "Expected eth0 UP, got DOWN"
        tc.result = [fail]

        skip = Skipped("No GPU available")
        skip.text = "Test requires a GPU device"
        tc.result = [skip]

Full example
------------

.. code-block:: python

        from junitparser import TestCase, TestSuite, JUnitXml, Failure, Skipped

        passed = TestCase("test_boot", "coreos.tests", 12.5)
        passed.system_out = "Boot completed"

        failed = TestCase("test_network", "coreos.tests", 30.0)
        f = Failure("Network unreachable"); f.text = "eth0 was DOWN"
        failed.result = [f]

        skipped = TestCase("test_gpu", "coreos.tests", 0.0)
        skipped.result = [Skipped("No GPU available")]

        suite = TestSuite("kola")
        suite.add_testcases([passed, failed, skipped])   # add many at once

        xml = JUnitXml()
        xml.add_testsuite(suite)
        xml.write("test-results.xml")

Quick reference
---------------

.. list-table::
   :header-rows: 1

   * - What
     - How
   * - Create test case
     - ``TestCase(name, classname, time)``
   * - stdout / stderr
     - ``tc.system_out = ...`` / ``tc.system_err = ...``
   * - Mark failed / skipped
     - ``tc.result = [Failure("msg")]`` / ``[Skipped("msg")]``
   * - Result details
     - ``f = Failure("msg"); f.text = "..."``
   * - Add one / many cases
     - ``suite.add_testcase(tc)`` / ``suite.add_testcases([...])``
   * - Write to file / stdout
     - ``xml.write("path.xml")`` / ``xml.write("/dev/stdout")``

vs the older ``python-junit-xml``
---------------------------------

.. list-table::
   :header-rows: 1

   * - What
     - ``python-junit-xml`` (old)
     - ``junitparser`` (new)
   * - TestCase
     - ``TestCase(name, classname, time, stdout, stderr)``
     - ``TestCase(name, classname, time)`` then set ``.system_out``/``.system_err``
   * - Mark failed
     - ``tc.add_failure_info(message, output)``
     - ``tc.result = [Failure(message)]`` + ``.text``
   * - Build suite
     - ``TestSuite(name, cases_list)``
     - ``TestSuite(name)`` then ``add_testcases(list)``
   * - Write
     - ``TestSuite.to_file(fd, [suites])``
     - ``JUnitXml()`` + ``add_testsuite`` + ``write(path)``

Note: ``JUnitXml.add_testsuite()`` doesn't return ``self``, so it can't be
chained - use separate lines (build, add, write).
