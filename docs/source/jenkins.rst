Jenkins - concepts
==================

What is Jenkins?
----------------

Jenkins is a Java-based automation server. It runs **jobs** - units of work
like building code, running tests, or deploying artifacts. Jobs are triggered
by events (git push, PR opened, timer, manual click) and run on **agents**
(machines that execute the work). The Jenkins **controller** orchestrates
everything.

Core concepts
-------------

Jobs
^^^^

A job is a configured task. Types include:

- **Freestyle** - simple, UI-configured build/test/deploy
- **Pipeline** - defined by a Jenkinsfile (config as code)
- **Multibranch pipeline** - one pipeline per branch/PR, auto-discovered from
  a source (GitHub, Git, …)
- **Folder** - groups jobs together
- **Seed job** - a job that creates OTHER jobs (via the Job DSL plugin)

Pipelines, agents, plugins
^^^^^^^^^^^^^^^^^^^^^^^^^^^

A pipeline is a series of **stages** (logical groups like "Build", "Test")
containing **steps** (``sh``, ``checkout``, ``archiveArtifacts``). Pipelines
run on **nodes** (agents) - physical, VM, container, or Kubernetes pods,
connected via SSH or JNLP. Jenkins is minimal out of the box; almost everything
comes from **plugins** (Job DSL, GitHub Branch Source, Pipeline, Branch API…).

Jenkinsfiles
------------

A Jenkinsfile is a Groovy script that defines a pipeline, stored in the repo
alongside the code it builds. Two flavors:

**Scripted** - raw Groovy, full language power, older style:

.. code-block:: groovy

        node {
            stage('Build') {
                checkout scm
                sh 'make build'
            }
            stage('Test') {
                sh 'make test'
            }
        }

**Declarative** - structured syntax, easier to read, more guardrails:

.. code-block:: groovy

        pipeline {
            agent any
            stages {
                stage('Build') { steps { sh 'make build' } }
                stage('Test')  { steps { sh 'make test'  } }
            }
        }

Jenkins configuration - XML under the hood
------------------------------------------

Every job is stored as an XML file on disk; the UI is just a form editor for it.
You can view/edit the XML directly:

- UI: job page → Configure
- REST API: ``curl <jenkins>/job/<name>/config.xml``
- On disk: ``$JENKINS_HOME/jobs/<name>/config.xml``

Job DSL plugin
--------------

Job DSL lets you define jobs as Groovy code instead of clicking through the UI.
The plugin translates DSL calls into Jenkins XML.

.. code-block:: groovy

        multibranchPipelineJob('afterburn') {
            branchSources {
                branchSource {
                    source {
                        github {
                            repoOwner('coreos')
                            repository('afterburn')
                        }
                    }
                }
            }
        }

A **seed job** (a regular pipeline/freestyle job) runs these DSL scripts via
``jobDsl`` steps.

Static vs Dynamic DSL
^^^^^^^^^^^^^^^^^^^^^^

- **Static DSL** - hand-written Groovy context classes in the job-dsl-plugin
  repo (``github {}``, ``git {}``…). Always available.
- **Dynamic DSL** - auto-generated at runtime from installed plugins via Java
  reflection. Only available inside Jenkins; tagged purple "Dynamic" in the API
  viewer at ``<jenkins>/plugin/job-dsl/api-viewer/index.html``.
- **configure block** - escape hatch for raw XML when neither exposes what you
  need. Fragile, no validation.

Constructors and setters (Java basics)
--------------------------------------

Reading plugin source means reading Java. A **constructor** runs once when an
object is created; its params are **required**. A **setter** configures an
optional property *after* creation.

.. code-block:: java

        public class GitHubSCMSource {
            public GitHubSCMSource(String repoOwner, String repository,
                                   String repositoryUrl, boolean configuredByUrl) {
                this.repoOwner = repoOwner;      // required to create the object
                this.repository = repository;
            }
            public void setCredentialsId(String credentialsId) {
                this.credentialsId = credentialsId;   // optional - skip for null
            }
        }

For Job DSL: constructor params become **required** DSL methods; setter methods
become **optional** ones.

Stapler annotations
-------------------

Plugins use the **Stapler** framework. Three annotations control how classes
become configurable:

- ``@DataBoundConstructor`` - marks the constructor whose params come from the
  form / DSL (one per class, all required).
- ``@DataBoundSetter`` - marks setter methods for optional params.
- ``@Symbol`` - gives a class a short DSL name; when present it *replaces* the
  camelCase class name entirely.

A class must implement ``Describable`` (usually via ``AbstractDescribableImpl``)
for the dynamic DSL to generate methods for it.

How the dynamic DSL generates methods
-------------------------------------

For each ``Describable`` class with ``@DataBoundConstructor``, the plugin
generates a DSL method:

1. **Method name** = ``@Symbol`` value if present, else camelCase class name.
2. **Required properties** = ``@DataBoundConstructor`` params.
3. **Optional properties** = ``@DataBoundSetter`` methods.
4. **Nesting** = if a param/setter type is itself ``Describable``, it becomes a
   nested closure.

The ``@ContextType`` annotation on a DSL context class tells the dynamic DSL
which ``Describable`` types to look for in that context.

DSL calls are nested closures, not dot-chains
----------------------------------------------

.. code-block:: groovy

        multibranchPipelineJob('afterburn') {   // delegate: MultibranchWorkflowJob
            branchSources {                      // delegate: BranchSourcesContext
                branchSource {                   // delegate: BranchSource (dynamic)
                    source {                     // delegate: SCMSource context
                        github {                 // delegate: GitHubSCMSource context
                            configuredByUrl(false)
                        }
                    }
                }
            }
        }

Each ``{}`` is a new scope with its own **delegate**. When you call
``configuredByUrl(false)`` inside ``github {}``, Groovy resolves it against the
closure's delegate (``Closure.DELEGATE_FIRST``): owner first? no - delegate
(the dynamic context for ``GitHubSCMSource``)? yes, it records the value. You
can't write ``github.configuredByUrl(false)`` because ``github`` isn't an object
you hold - it's a method that takes a closure. **The nesting IS the structure:**
moving a call to the wrong level calls it on the wrong delegate and either
silently does nothing or throws ``MissingMethodException``.

The configure block (raw XML)
-----------------------------

.. code-block:: groovy

        configure {
            it / sources / data / 'jenkins.branch.BranchSource' / strategy { ... }
        }

- ``it`` = root XML node of the job config
- ``/`` = navigate to (or create) a child element, like XPath
- ``{ ... }`` and ``<<`` = APPEND children to the found node

**Pitfall:** ``/`` returns the existing node if found, and closures *append*
rather than replace - if the static DSL already created a ``<traits>`` node and
your configure block appends another, you get duplicate XML.

Multibranch pipeline - key concepts
-----------------------------------

A multibranch pipeline auto-creates a sub-job for each branch/PR. Four layers:

- **Branch sources** - which repo to scan and with what credentials
  (``GitHubSCMSource``, ``GitSCMSource``).
- **Traits** - what to discover: ``BranchDiscoveryTrait`` (regular branches),
  ``OriginPullRequestDiscoveryTrait`` (same-repo PRs),
  ``ForkPullRequestDiscoveryTrait`` (fork PRs, with a security-critical
  ``trust`` policy: ``TrustNobody`` / ``TrustContributors`` / ``TrustPermission``
  / ``TrustEveryone``).
- **Strategy** - a ``BranchPropertyStrategy`` applies properties to discovered
  branches (e.g. ``DurabilityHintBranchProperty``).
- **Build strategies** - when to build. ``ChangeRequestBuildStrategyImpl``
  (``buildChangeRequests``) has ``ignoreTargetOnlyChanges`` and
  ``ignoreUntrustedChanges``.

Migrating a configure block to native DSL
-----------------------------------------

A repeatable approach when replacing a fragile ``configure`` block:

1. **Understand what the old block patches** - e.g. the traits, strategy and
   buildStrategies XML sections.
2. **Check the static DSL** - read the job-dsl-plugin source; a helper like
   ``github {}`` may expose trait flags but not ``strategy``/``buildStrategies``
   if its context isn't extensible.
3. **Check the dynamic DSL** - confirm the class is ``Describable`` with the
   relevant ``@DataBoundSetter`` methods; if so, ``branchSource {}`` gives full
   access.
4. **Mind parameter differences** - the dynamic ``github {}`` (inside
   ``source {}``) differs from the static one (inside ``branchSources {}``):
   ``credentialsId`` vs ``checkoutCredentialsId``, traits set directly rather
   than via boolean flags, and a constructor requiring ``repositoryUrl`` /
   ``configuredByUrl``.
