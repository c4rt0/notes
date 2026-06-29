YAML config as nested maps (Python + Groovy)
============================================

Loading a YAML file gives you nested dictionaries (Python) or maps (Groovy).
The concepts are identical; only the syntax differs. Examples use a config
shaped like the Fedora CoreOS pipeline:

.. code-block:: yaml

        ocp_node_builds:
          release:
            "4.22-9.8":
              source_config:
                url: https://github.com/openshift/rhel-coreos-config
                ref: release-4.22-9.8
              from: quay.io/coreos/rhel-coreos-base:9.8
              build_args_file: true
              arches: [x86_64, aarch64]

Python
------

.. code-block:: python

        import yaml
        config = yaml.safe_load(open("config.yaml"))

        config["ocp_node_builds"]["release"]["4.22-9.8"]   # bracket access only
        release = "4.22-9.8"
        info = config["ocp_node_builds"]["release"][release]  # key from a variable
        info["source_config"]["url"]
        info["arches"][0]                                  # lists → Python lists

Missing keys raise ``KeyError`` - use ``.get()`` for safe access:

.. code-block:: python

        info.get("skip_brew_upload")          # → None (no error)
        info.get("skip_brew_upload", False)   # → explicit default
        info.get("yumrepo", {}).get("url")    # → None if either level is missing

.. list-table::
   :header-rows: 1

   * - YAML
     - Python type
     - Access
   * - mapping (``key: val``)
     - ``dict``
     - ``d['key']`` / ``d.get('key')``
   * - sequence (``- item``)
     - ``list``
     - ``d[0]`` / ``for x in d`` / ``len(d)``
   * - scalar
     - ``str``/``int``/``bool``
     - direct value
   * - missing key
     - ``KeyError``
     - ``.get(key)`` / ``.get(key, default)``

Groovy
------

In a Jenkins pipeline, ``readYaml`` returns plain Groovy maps and lists. Dot and
bracket notation are interchangeable; brackets are needed for variable keys.

.. code-block:: groovy

        def pipecfg = readYaml file: 'config.yaml'

        pipecfg.ocp_node_builds.release."4.22-9.8"
        // params.RELEASE is a string like "4.22-9.8" chosen at build time:
        def info = pipecfg.ocp_node_builds.release[params.RELEASE]
        info.source_config.url
        info.from
        info.arches[0]; info.arches.size()

Missing keys are ``null`` (not an error), which is why opt-in flags work without
defaults. Use ``?:`` for a default and ``?.`` for safe navigation:

.. code-block:: groovy

        info.skip_brew_upload            // → null (not in YAML)
        if (info.skip_brew_upload) { }   // null is falsy → safely skipped
        def skip = info.skip_brew_upload ?: false   // default

        info.yumrepo.url                 // → NullPointerException if yumrepo absent
        info.yumrepo?.url                // → null (safe)

Python vs Groovy side-by-side
-----------------------------

.. list-table::
   :header-rows: 1

   * - Operation
     - Python
     - Groovy
   * - Load YAML
     - ``yaml.safe_load(open(f))``
     - ``readYaml file: f``
   * - Access key
     - ``d['key']``
     - ``d.key`` or ``d['key']``
   * - Missing key
     - ``KeyError`` (crash)
     - ``null`` (silent)
   * - Safe with default
     - ``d.get('key', False)``
     - ``d.key ?: false``
   * - Safe nested access
     - ``d.get('a', {}).get('b')``
     - ``d.a?.b``
   * - Iterate / length
     - ``for x in lst`` / ``len(lst)``
     - ``for (x in lst)`` / ``lst.size()``
   * - String interpolation
     - ``f"hello {name}"``
     - ``"hello ${name}"``
