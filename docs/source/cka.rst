CKA preparation
===============

Practical notes for the Certified Kubernetes Administrator exam. The exam is
hands-on in a live cluster, so it's all ``kubectl``. The official
`kubernetes.io/docs <https://kubernetes.io/docs/>`_ is allowed during the exam -
practise finding YAML snippets there fast.

Exam setup (do this first)
--------------------------

.. code-block:: bash

        alias k=kubectl
        export do="--dry-run=client -o yaml"   # generate manifests, don't apply
        export now="--force --grace-period=0"  # delete pods immediately
        # always switch context before a task:
        kubectl config use-context <cluster>

Generate manifests imperatively, then edit - far faster than writing YAML from
scratch:

.. code-block:: bash

        k run nginx --image=nginx $do > pod.yaml
        k create deploy web --image=nginx --replicas=3 $do > deploy.yaml
        k create svc clusterip web --tcp=80:80 $do > svc.yaml
        k explain pod.spec.containers          # field reference, offline

Cluster architecture & RBAC
---------------------------

**Create a role and bind it to a service account** (least-privilege access):

.. code-block:: bash

        k create sa deploy-bot
        k create role pod-reader --verb=get,list,watch --resource=pods
        k create rolebinding read-pods --role=pod-reader --serviceaccount=default:deploy-bot
        # cluster-wide equivalents: clusterrole / clusterrolebinding
        k auth can-i list pods --as=system:serviceaccount:default:deploy-bot

Control-plane components run as static pods in ``/etc/kubernetes/manifests/`` on
the control-plane node (kubelet watches that dir). Kubeconfigs live in
``/etc/kubernetes/*.conf``.

etcd backup and restore
----------------------

A classic exam task. Snapshot and restore with ``etcdctl`` (API v3):

.. code-block:: bash

        ETCDCTL_API=3 etcdctl snapshot save /opt/snap.db \
          --endpoints=https://127.0.0.1:2379 \
          --cacert=/etc/kubernetes/pki/etcd/ca.crt \
          --cert=/etc/kubernetes/pki/etcd/server.crt \
          --key=/etc/kubernetes/pki/etcd/server.key

        ETCDCTL_API=3 etcdctl snapshot restore /opt/snap.db \
          --data-dir=/var/lib/etcd-restore
        # then point etcd's static pod manifest at the new --data-dir and restart

Cluster upgrade (kubeadm)
------------------------

Upgrade the control plane first, then each worker. Always drain before, uncordon
after:

.. code-block:: bash

        # control plane
        kubectl drain cp --ignore-daemonsets
        apt-get install -y kubeadm=1.30.x   # or dnf
        kubeadm upgrade plan
        kubeadm upgrade apply v1.30.x
        apt-get install -y kubelet=1.30.x kubectl=1.30.x
        systemctl daemon-reload && systemctl restart kubelet
        kubectl uncordon cp
        # workers: kubeadm upgrade node (instead of apply)

Workloads & scheduling
---------------------

.. code-block:: bash

        k scale deploy web --replicas=5
        k set image deploy/web nginx=nginx:1.27   # rolling update
        k rollout status deploy/web
        k rollout undo deploy/web                  # roll back

        k create configmap app-cfg --from-literal=LOG_LEVEL=debug
        k create secret generic db --from-literal=password=changeme

Consume config/secrets, set resource requests/limits, and place pods with
``nodeSelector`` / affinity / taints:

.. code-block:: yaml

        spec:
          containers:
            - name: app
              image: nginx
              envFrom:
                - configMapRef: { name: app-cfg }
              resources:
                requests: { cpu: "100m", memory: "128Mi" }
                limits:   { cpu: "250m", memory: "256Mi" }
          nodeSelector:
            disktype: ssd

.. code-block:: bash

        k taint node worker1 key=value:NoSchedule   # repel pods without a matching toleration
        k label node worker1 disktype=ssd

A pod tolerates a taint with:

.. code-block:: yaml

        tolerations:
          - key: "key"
            operator: "Equal"
            value: "value"
            effect: "NoSchedule"

Services & networking
--------------------

.. code-block:: bash

        k expose deploy web --port=80 --target-port=8080     # ClusterIP
        k expose deploy web --type=NodePort --port=80         # NodePort
        k get endpoints web                                   # verify it found pods

Ingress routes external HTTP to services; NetworkPolicies restrict pod traffic
(default is allow-all until a policy selects a pod). A deny-all-ingress policy:

.. code-block:: yaml

        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata: { name: deny-ingress }
        spec:
          podSelector: {}        # all pods in the namespace
          policyTypes: [Ingress]

DNS: services resolve as ``<svc>.<namespace>.svc.cluster.local`` via CoreDNS
(check the ``coredns`` pods in ``kube-system`` if name resolution fails).

Storage
-------

A PersistentVolumeClaim binds to a PersistentVolume (or is provisioned
dynamically by a StorageClass). Access modes: ``ReadWriteOnce`` (one node),
``ReadOnlyMany``, ``ReadWriteMany``.

.. code-block:: yaml

        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata: { name: data }
        spec:
          accessModes: [ReadWriteOnce]
          storageClassName: standard
          resources:
            requests: { storage: 1Gi }

Mount it in a pod via ``volumes`` (``persistentVolumeClaim.claimName``) and
``volumeMounts``.

Troubleshooting
--------------

.. code-block:: bash

        k get pods -A -o wide              # what's where, what's failing
        k describe pod <p>                 # events at the bottom - start here
        k logs <p> [-c container] [--previous]
        k get events --sort-by=.metadata.creationTimestamp
        k exec -it <p> -- sh

        # node down?
        k get nodes
        ssh <node>; systemctl status kubelet; journalctl -u kubelet
        # control-plane pod down? check the static manifests + container runtime:
        crictl ps -a | grep -i kube

Common causes: ``ImagePullBackOff`` (wrong image/secret), ``CrashLoopBackOff``
(app exits - read logs), ``Pending`` (no schedulable node: resources, taints,
or PVC unbound), ``kubelet`` not running on a ``NotReady`` node.

Exam tips
---------

- **Switch context first** on every question (``kubectl config use-context``) -
  a correct answer in the wrong cluster scores zero.
- Generate YAML with ``$do`` and edit; don't hand-write manifests.
- Use ``k explain <resource>.<field>`` and the docs site instead of memorizing
  every field.
- Delete stuck pods fast with ``$now`` (``--force --grace-period=0``).
- Don't get stuck: flag hard questions, finish the cheap ones, come back.
