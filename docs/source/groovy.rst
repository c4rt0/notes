Groovy - concepts
=================

Groovy is a dynamic language for the JVM. It compiles to Java bytecode and can
call any Java class directly - scripting-language ergonomics with full access
to the Java ecosystem.

.. code-block:: groovy

        // Java needs ~6 lines; Groovy:
        def names = ["alice", "bob"]
        names.each { println it.toUpperCase() }

Why it matters here: all Jenkins pipeline code is Groovy - Jenkinsfiles, Job DSL
scripts, and shared libraries. When you see ``node {}``, ``stage {}``,
``sh 'make build'``, that's Groovy, not a custom language.

Types and variables
--------------------

.. code-block:: groovy

        def x = 42          // inferred Integer
        def s = "hello"     // inferred String
        String name = "alice"   // explicit types work too
        int count = 5

``def`` means "infer the type." Everything is an object - no primitives
(``int`` auto-boxes to ``Integer``), and ``null`` is valid for any type. In
Jenkinsfiles ``def`` is common; in shared libraries explicit types read better.

Strings - GString vs String
---------------------------

The single most important Groovy concept for Jenkins work - getting it wrong
causes injection bugs.

.. code-block:: groovy

        def name = 'alice'
        'hello ${name}'    // single quotes: literal text "hello ${name}"
        "hello ${name}"    // double quotes: interpolated "hello alice"  (a GString)

In Jenkinsfiles, ``sh`` steps with double quotes can be injection vectors:

.. code-block:: groovy

        // DANGEROUS - if the branch name contains '; rm -rf /' it executes
        sh "echo Building ${env.BRANCH_NAME}"
        // SAFE - no interpolation, Jenkins expands the env var
        sh 'echo Building $BRANCH_NAME'

Other forms: triple quotes (``"""..."""`` / ``'''...'''``) for multiline, and
slashy strings (``/\d+\.\d+/``) for regex without double-escaping.

Closures
--------

Closures are the core of how Jenkins DSL works - every ``{ ... }`` block is one.

.. code-block:: groovy

        def greet = { name -> println "hello ${name}" }
        greet("alice")
        def greet2 = { println "hello ${it}" }   // single param: use `it`

When the last argument to a method is a closure, it can go *outside* the
parentheses - which is why DSL looks like configuration but is really code:

.. code-block:: groovy

        list.each { println it }          // == list.each({ println it })

        multibranchPipelineJob('my-job') {  // a method call with a closure arg
            branchSources { /* ... */ }
        }

Delegation
^^^^^^^^^^

A closure has a **delegate** - an object whose methods it can call as if its
own. This is how DSL blocks work:

.. code-block:: groovy

        def config = new GitHubConfig()
        def closure = {
            repoOwner('coreos')      // calls config.repoOwner('coreos')
            repository('afterburn')
        }
        closure.delegate = config
        closure.resolveStrategy = Closure.DELEGATE_FIRST
        closure()

When you write ``github { repoOwner('coreos') }`` in Job DSL, the plugin creates
a context object, sets it as the closure's delegate, and runs the closure so the
calls hit the context's methods. Nested blocks are closures-within-closures,
each delegated to a different context object.

Operators
---------

.. code-block:: groovy

        person?.address?.city       // ?.  safe dereference (null if any part null)
        names*.toUpperCase()        // *.  spread over a collection
        userName ?: 'anonymous'     // ?:  Elvis - default if null/false

In ``configure`` blocks the ``Node`` class overloads ``/`` (navigate to/create a
child element, XPath-style) and ``<<`` (append children). **Pitfall:** ``<<``
always *appends* - if the parent already has that child, you get a duplicate
(a common cause of duplicate-element bugs).

Collections
-----------

.. code-block:: groovy

        def list = [1, 2, 3]
        list << 4                       // append
        list[-1]                        // last element
        list[1..2]                      // range slice
        list.collect { it * 2 }         // map
        list.findAll { it > 2 }         // filter

        def map = [name: 'alice', age: 30]
        map.name                        // property access
        map['name']                     // subscript access

**Gotcha:** map keys are literal strings by default. To use a variable's value
as a key, wrap it in parentheses: ``[(key): 'alice']``.

Control flow
------------

``if/else``, ``for (item in list)`` and ``while`` work as in Java. Groovy truth
makes more things falsy: ``null``, ``""``, ``0``, ``[]`` and ``[:]`` are all
false. ``switch`` matches on types, ranges, regex and closures:

.. code-block:: groovy

        switch (x) {
            case String:        println "a string"; break
            case 1..10:         println "1 to 10";  break
            case ~/\d+/:        println "digits";   break
            case { it > 100 }:  println "big";      break
        }

Methods and classes
-------------------

.. code-block:: groovy

        String greet(String name) {
            "hello ${name}"          // last expression is the implicit return
        }

        class Person {
            String name              // auto getter/setter
            int age
            String toString() { "${name} (${age})" }
        }
        def p = new Person(name: 'alice', age: 30)   // map-based constructor
        println p.name                                // calls getName()

Groovy also has its own ``trait`` concept (interfaces with implementations) -
unrelated to Jenkins ``SCMSourceTrait``, same word, different thing.

Groovy vs Java - quick reference
--------------------------------

.. list-table::
   :header-rows: 1

   * - Feature
     - Java
     - Groovy
   * - Semicolons
     - required
     - optional
   * - Type declarations
     - required
     - optional (``def``)
   * - String interpolation
     - no
     - ``"hello ${name}"``
   * - Closures
     - lambdas (limited)
     - full closures with delegate
   * - Property access
     - ``obj.getName()``
     - ``obj.name``
   * - Null safety
     - manual checks
     - ``?.`` operator
   * - Regex
     - ``Pattern.compile("\\d+")``
     - ``~/\d+/``
   * - List / Map literal
     - ``Arrays.asList(...)``
     - ``[1, 2, 3]`` / ``[k: "v"]``

Common gotchas
--------------

- **GString in maps** - ``["${key}": "alice"]`` makes the key a *GString*, so
  ``map["name"]`` returns null (String != GString). Use ``[(key): "alice"]``.
- **== vs .is()** - in Groovy ``==`` calls ``.equals()`` (value comparison);
  use ``.is()`` for reference identity.
- **method/property missing** - Groovy resolves dynamically; a missing call
  routes through ``methodMissing()``/``propertyMissing()`` on the delegate,
  which is exactly how DSL blocks work.
- **Escaping in triple-quoted GStrings** - in Job DSL seed scripts the DSL code
  lives in ``"""..."""``; ``${name}`` interpolates from the outer scope, while
  ``\$`` produces a literal ``$`` in the generated code.
