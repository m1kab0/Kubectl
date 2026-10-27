# Kubernetes Components

A quick reference of the main Kubernetes building blocks: what each one is and what it's used for.

---

## 1. Control Plane Components

The control plane is the "brain" of the cluster. It decides what should run where and keeps the cluster in the state you asked for.

| Component | What it does | Used for |
|---|---|---|
| **kube-apiserver** | The front door of the cluster. Every request (from `kubectl`, the UI, or other components) goes through its REST API. | Validating and processing all changes to the cluster; the only component that talks to etcd directly. |
| **etcd** | A consistent, distributed key-value store. | Storing the whole cluster state: every object, config, and status. Back it up. |
| **kube-scheduler** | Watches for new Pods with no node assigned and picks a node for each one. | Placing workloads based on resources, affinity/anti-affinity, taints/tolerations, and other constraints. |
| **kube-controller-manager** | Runs the built-in controllers (Node, ReplicaSet, Deployment, Job, EndpointSlice, ServiceAccount, and others). | Constantly comparing the actual state with the desired state and fixing the differences. |
| **cloud-controller-manager** | Runs the controllers that talk to a cloud provider (AWS, GCP, Azure, ...). | Creating cloud load balancers, managing routes, and noticing when cloud VMs (nodes) are deleted. Optional when not on a cloud. |

---

## 2. Worker Node Components

Worker nodes are the machines (VMs or physical servers) that actually run your applications.

| Component | What it does | Used for |
|---|---|---|
| **kubelet** | An agent on every node that takes Pod specs from the API server and makes sure those containers are running and healthy. | Starting and stopping containers, running liveness/readiness probes, and reporting node and Pod status. |
| **kube-proxy** | Maintains network rules on each node (iptables, IPVS, or nftables). | Making Services work by routing traffic sent to a Service IP to the right Pods. Some CNIs, such as Cilium, can replace it. |
| **Container runtime** | The software that actually runs containers (containerd, CRI-O). Uses the CRI interface. | Pulling images and running containers. |

---

## 3. Add-ons (cluster-level features)

| Add-on | What it does | Used for |
|---|---|---|
| **CNI plugin** (Calico, Cilium, Flannel, ...) | Implements Pod networking. | Giving every Pod an IP address and letting Pods talk to each other across nodes; enforcing NetworkPolicies. |
| **CoreDNS** | The cluster DNS server. | Service discovery by name, e.g. `my-svc.my-namespace.svc.cluster.local`. |
| **Metrics Server** | Collects CPU and memory usage from kubelets. | `kubectl top` and the Horizontal Pod Autoscaler. |
| **Dashboard** | A web UI. | Viewing and managing cluster resources in a browser. |
| **Ingress controller** (NGINX, Traefik, ...) | Implements Ingress / Gateway API rules. | HTTP(S) routing from outside the cluster to Services. |

---

## 4. Workload Resources

| Resource | What it is | Used for |
|---|---|---|
| **Pod** | The smallest deployable unit: one or more containers that share a network (IP) and storage. | Running your containers. Usually created by higher-level controllers, not by hand. |
| **ReplicaSet** | Keeps a specified number of identical Pod replicas running. | Self-healing and scaling. Normally managed by a Deployment. |
| **Deployment** | Manages ReplicaSets declaratively. | Stateless apps: rolling updates, rollbacks, and scaling. The most common workload. |
| **StatefulSet** | Like a Deployment, but each Pod gets a stable name (`db-0`, `db-1`) and its own persistent storage. | Databases and clustered apps (PostgreSQL, Kafka, Elasticsearch). |
| **DaemonSet** | Runs one copy of a Pod on every node (or on a chosen set of nodes). | Node agents: log collectors, monitoring exporters, CNI or storage daemons. |
| **Job** | Runs Pods until they finish successfully. | One-off tasks: database migrations, batch processing. |
| **CronJob** | Creates Jobs on a schedule (cron syntax). | Recurring tasks: backups, reports, cleanups. |

---

## 5. Networking Resources

| Resource | What it is | Used for |
|---|---|---|
| **Service** | A stable virtual IP and DNS name in front of a set of Pods (chosen by labels). | Load-balancing traffic to Pods whose IPs keep changing. |
| ↳ **ClusterIP** | The default Service type. Reachable only inside the cluster. | Internal communication between apps. |
| ↳ **NodePort** | Opens a port (30000–32767) on every node. | Simple external access, testing. |
| ↳ **LoadBalancer** | Provisions an external cloud load balancer. | Exposing an app to the internet on a cloud. |
| ↳ **ExternalName** | A DNS CNAME to an external hostname. | Pointing to services outside the cluster. |
| ↳ **Headless** (`clusterIP: None`) | No virtual IP; DNS returns the individual Pod IPs. | StatefulSets and direct Pod discovery. |
| **Ingress** | HTTP/HTTPS routing rules (host or path → Service). Needs an Ingress controller. | One entry point for many services, TLS termination. |
| **Gateway API** (Gateway, HTTPRoute, ...) | The newer, more expressive successor to Ingress. | Advanced traffic routing, role-oriented configuration. |
| **EndpointSlice** | Tracks the actual Pod IPs behind a Service. | Used internally by kube-proxy and DNS. |
| **NetworkPolicy** | Firewall rules for Pods (ingress/egress). | Restricting which Pods can talk to which. |

---

## 6. Configuration & Storage

| Resource | What it is | Used for |
|---|---|---|
| **ConfigMap** | Non-sensitive key-value configuration. | Injecting env vars or config files into Pods. |
| **Secret** | Sensitive data (base64-encoded, can be encrypted at rest). | Passwords, API tokens, TLS certificates. |
| **Volume** | Storage attached to a Pod (emptyDir, hostPath, configMap, ...). | Sharing files between containers; data that outlives a container restart. |
| **PersistentVolume (PV)** | A piece of storage in the cluster (a disk, NFS, cloud volume). | Durable storage that is independent of any Pod's lifecycle. |
| **PersistentVolumeClaim (PVC)** | A Pod's request for storage (size and access mode). | Letting apps ask for storage without knowing the backend. |
| **StorageClass** | Defines a "type" of storage and its provisioner. | Dynamic provisioning: PVs are created automatically when a PVC is made. |
| **CSI driver** | A plugin interface for storage vendors. | Connecting Kubernetes to EBS, Ceph, Longhorn, and other storage. |

---

## 7. Cluster Organization & Security

| Resource | What it is | Used for |
|---|---|---|
| **Namespace** | A virtual partition of the cluster. | Separating teams or environments (dev/staging/prod) and scoping names and quotas. |
| **Labels & Selectors** | Key-value tags on objects. | Grouping and selecting objects (how Services find Pods). |
| **Annotations** | Non-identifying metadata. | Tooling config, notes, build info. |
| **ServiceAccount** | An identity for processes running in Pods. | Letting Pods authenticate to the API server. |
| **Role / ClusterRole** | A set of permissions (verbs on resources), within a namespace or cluster-wide. | Defining what someone is allowed to do (RBAC). |
| **RoleBinding / ClusterRoleBinding** | Grants a Role to users, groups, or ServiceAccounts. | Assigning permissions. |
| **ResourceQuota** | Limits total resource use per namespace. | Preventing one team from using up the cluster. |
| **LimitRange** | Default and maximum CPU/memory per Pod or container in a namespace. | Enforcing sane resource requests and limits. |
| **Pod Security Admission** | Enforces the Pod Security Standards (privileged / baseline / restricted). | Blocking insecure Pod configurations. |

---

## 8. Scaling & Availability

| Resource | What it is | Used for |
|---|---|---|
| **HorizontalPodAutoscaler (HPA)** | Changes the number of replicas based on metrics (CPU, memory, custom). | Scaling out under load. |
| **VerticalPodAutoscaler (VPA)** | Adjusts a Pod's CPU/memory requests (add-on). | Right-sizing containers. |
| **Cluster Autoscaler / Karpenter** | Adds or removes nodes (add-on). | Scaling the cluster itself. |
| **PodDisruptionBudget (PDB)** | The minimum number of Pods that must stay available during voluntary disruptions. | Safe node drains and upgrades. |
| **Probes** (liveness, readiness, startup) | Health checks run by the kubelet. | Restarting unhealthy containers and only sending traffic to ready ones. |

---

## 9. Extending Kubernetes

| Concept | What it is | Used for |
|---|---|---|
| **CustomResourceDefinition (CRD)** | Adds your own resource types to the API. | Defining new objects (for example `Certificate` or `PostgresCluster`). |
| **Operator** | A custom controller plus CRDs that encode operational knowledge. | Automating complex apps (backups, failover, upgrades). |
| **Admission webhooks** | Validating or mutating hooks on API requests. | Policy enforcement (OPA Gatekeeper, Kyverno), sidecar injection. |

---

## How it fits together (the request flow)

1. You run `kubectl apply -f deployment.yaml`, which sends the request to the **kube-apiserver**.
2. The API server validates the object and stores it in **etcd**.
3. The **Deployment controller** creates a **ReplicaSet**, and the ReplicaSet controller creates the **Pods**.
4. The **scheduler** assigns each Pod to a node.
5. On that node, the **kubelet** asks the **container runtime** to pull the image and start the containers.
6. The **CNI** gives the Pod an IP address. A **Service** plus **kube-proxy** route traffic to it, and **CoreDNS** makes it reachable by name.
7. The controllers keep watching. If a Pod dies, a new one is created automatically.
