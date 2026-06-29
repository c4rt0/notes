Java - concepts
===============

Why Java matters here: Jenkins and all its plugins are written in Java. You
rarely write Java day-to-day, but you read Java plugin source on GitHub to
learn what DSL methods exist, what parameters they take, and how they work.
Groovy runs on the JVM and can call any Java class directly.

Annotations - metadata on classes and methods
----------------------------------------------

Annotations are markers (``@Name``) attached to classes, methods or fields.
They look like Python decorators but work differently - they're **passive**
metadata, read later via reflection, rather than wrapping the target.

.. code-block:: java

        @Symbol("gitHubBranchDiscovery")
        public static class DescriptorImpl extends SCMSourceTraitDescriptor { }

        public @interface Symbol { String[] value(); }   // declares the annotation

        // read at runtime via reflection:
        Symbol s = clazz.getAnnotation(Symbol.class);
        if (s != null) { String name = s.value()[0]; }

This is exactly what the Job DSL plugin does - it scans installed plugins for
``@DataBoundConstructor`` and ``@Symbol`` via reflection and auto-generates DSL
methods.

.. list-table::
   :header-rows: 1

   * - Annotation
     - What it marks
     - Who reads it
   * - ``@DataBoundConstructor``
     - constructor with required params
     - Stapler (UI), Job DSL
   * - ``@DataBoundSetter``
     - setter with optional param
     - Stapler, Job DSL
   * - ``@Symbol``
     - short name for a class
     - Job DSL, Pipeline DSL
   * - ``@Override`` / ``@Deprecated``
     - overrides parent / outdated
     - Java compiler (built-in)

Interfaces, inheritance, generics
---------------------------------

An **interface** defines methods a class must implement. In Jenkins,
``Describable`` is the key one - a class that implements it can have DSL methods
generated for it.

.. code-block:: java

        public class BranchSource extends AbstractDescribableImpl<BranchSource> {
            // extends = inherit from one parent class (single inheritance)
            // implements = fulfill one or more interfaces
        }

``AbstractDescribableImpl`` is a convenience base that implements ``Describable``
for you; most plugin classes extend it rather than implementing ``Describable``
directly. The ``<T>`` **generics** syntax (like Python's ``list[str]`` hints) is
a compile-time type constraint - when reading plugin code you can mostly ignore
it: ``AbstractDescribableImpl<BranchSource>`` just means "a Describable for
BranchSource".

Access modifiers
----------------

.. list-table::
   :header-rows: 1

   * - Modifier
     - Visible to
   * - ``public``
     - everything
   * - ``protected``
     - same class + subclasses
   * - (none)
     - same package only
   * - ``private``
     - same class only

In plugin source, ``public`` methods with ``@DataBoundConstructor`` /
``@DataBoundSetter`` are the ones that become DSL methods; ``private`` fields are
the internal state they set.

Reading Java plugin source - what to look for
---------------------------------------------

When checking whether a DSL method exists for a plugin class on GitHub:

1. **Find the class** - e.g. ``BranchSource.java`` in ``jenkinsci/branch-api-plugin``.
2. **Check** ``@DataBoundConstructor`` - its params become *required* DSL properties.
3. **Check** ``@DataBoundSetter`` - its methods become *optional* DSL properties.
4. **Check** ``@Symbol`` on the ``DescriptorImpl`` inner class - that's the DSL
   method name.
5. **Check** ``extends`` / ``implements`` - if it extends
   ``AbstractDescribableImpl``, the dynamic DSL can generate methods for it.
