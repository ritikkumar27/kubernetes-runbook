# 13 · Nodes and Cluster Admin

The machines themselves. Pod-level autoscaling adds pods; node-level autoscaling adds machines.

| Serial | Command |
|--------|---------|
| K8S-033 | `kubectl get nodes` |
| K8S-035 | `kubectl top node` |

## Node types

| Type | Runs | Notes |
|------|------|-------|
| Control plane | API server, scheduler, controller manager, etcd | Managed for you on GKE/EKS/AKS. Formerly called "master", that term is deprecated |
| Worker nodes | Your pods, kubelet, kube-proxy, container runtime | What "node" usually means on its own |

Managed offerings (GKE, EKS, AKS) can also autoscale the **nodes**, which is separate from the HPA: the HPA changes pod count inside a fixed set of machines, cluster autoscaling changes the number of machines.

## K8S-033 · `kubectl get nodes`

- **First seen:** [[Ch7 Scaling]] → Nodes
- **Purpose:** List the machines in the cluster and whether they can accept pods.

```sh
kubectl get nodes
```

Typical columns: `NAME  STATUS  ROLES  AGE  VERSION`. `STATUS Ready` means the node is healthy and schedulable.

**Flags**

| Flag | Effect |
|------|--------|
| `-o wide` | Adds internal/external IP, OS image, kernel, container runtime |
| `--show-labels` | Label columns, including `node-role.kubernetes.io/...` |
| `-L <label>` | Show one specific label as a column |
| `-o yaml` | Full object, including capacity/allocatable and conditions |

**Notes**

- `ROLES <none>` on a managed cluster is normal: the control plane is hidden, so the workers carry no role label.
- Nodes are **cluster-scoped**, so `-n` does nothing here.
- `Allocatable` (in `-o yaml`) is what the scheduler actually uses, and it is lower than `Capacity` because the OS and kubelet reserve some.
- To see why a pod does not fit, `kubectl describe node <name>` shows the sum of pod requests against allocatable, plus `Allocated resources` in percentages.
- Draining and cordoning (`kubectl drain`, `kubectl cordon`) are the maintenance commands you will need later. Not covered in the notes yet.
- `kubectl get nodes -o wide` plus `kubectl top node` is the usual pair for "is the cluster full, or is one node sick".

## K8S-035 · `kubectl top node`

- **First seen:** [[Ch9 Local Cluster]] → Metrics
- **Purpose:** Show CPU and memory usage per node, the machine-level counterpart to `kubectl top pod`.

```sh
kubectl top node
```

Typical columns: `NAME  CPU(cores)  CPU%  MEMORY(bytes)  MEMORY%`.

**Flags**

| Flag | Effect |
|------|--------|
| `--sort-by=cpu` / `--sort-by=memory` | Rank nodes by load |
| `--show-capacity` | Show allocatable capacity next to usage |
| `-l <selector>` | Limit to matching nodes |

**Notes**

- Requires metrics-server. On Minikube enable it with `minikube addons enable metrics-server`, then wait for that pod to become ready.
- `CPU%` and `MEMORY%` are measured against **allocatable**, not total capacity.
- Usage is not the same as scheduling headroom. A node at 10% usage can still refuse a pod because *requests* are what the scheduler counts. Use `kubectl describe node` for allocated requests.
- On a single-node Minikube cluster this command has one row, which makes it a quick way to confirm metrics are working at all.
