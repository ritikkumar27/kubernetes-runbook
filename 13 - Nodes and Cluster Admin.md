# 13 · Nodes and Cluster Admin

The machines themselves. Pod-level autoscaling adds pods; node-level autoscaling adds machines.

| Serial | Command |
|--------|---------|
| K8S-033 | `kubectl get nodes` |

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
