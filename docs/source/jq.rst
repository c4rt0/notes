jq & yq - JSON and YAML on the CLI
==================================

``jq`` filters and transforms JSON; ``yq`` does the same for YAML and converts
between the two. Install with ``dnf install -y jq`` (yq: ``pip install yq`` or the
mikefarah Go binary - syntax below is the mikefarah one).

jq basics
---------

.. code-block:: console

   $ echo '{"name":"fcos","ver":42}' | jq .       # pretty-print
   $ jq '.name' file.json                          # a field -> "fcos"
   $ jq -r '.name' file.json                       # raw (no quotes) -> fcos
   $ jq '.items[0].metadata.name' k8s.json         # nested + array index
   $ jq '.items[].metadata.name' k8s.json          # every item's name
   $ jq 'keys' file.json ; jq 'length' file.json

Selecting and filtering
-----------------------

.. code-block:: console

   $ jq '.pods[] | select(.status == "Running")' p.json
   $ jq '.[] | select(.age > 30) | .name' people.json
   $ jq 'map(select(.enabled)) | length' items.json   # count matching
   $ jq '.tags | contains(["prod"])' svc.json

Transforming
------------

.. code-block:: console

   $ jq '.items | map(.metadata.name)' k8s.json       # array of names
   $ jq '{name: .metadata.name, ns: .metadata.namespace}' pod.json   # reshape
   $ jq -r '.data | to_entries[] | "\(.key)=\(.value)"' cm.json      # key=value lines
   $ jq -s 'add' a.json b.json                        # slurp files, merge arrays

Building JSON from the shell
----------------------------

.. code-block:: console

   $ jq -n --arg name "$USER" --argjson n 3 '{user: $name, count: $n}'

yq (YAML)
---------

Same query language, for YAML - plus conversion:

.. code-block:: console

   $ yq '.spec.replicas' deploy.yaml
   $ yq '.services | keys' docker-compose.yaml
   $ yq -i '.spec.replicas = 3' deploy.yaml           # edit in place
   $ yq -o=json '.' config.yaml                        # YAML -> JSON
   $ kubectl get pod x -o json | yq -P                 # JSON -> pretty YAML
