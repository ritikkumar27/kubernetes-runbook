# 01 · Cluster and Client Info

Commands that tell you **what you're talking to** and **what it offers**.

| Serial | Command |
|--------|---------|
| K8S-001 | `kubectl version` |

## K8S-001 · `kubectl version`

- **First seen:** [[Ch1 Pods - 1]] → Welcome to "Learn Kubernetes"
- **Purpose:** Print the kubectl client version and the cluster's server version. First sanity check on any cluster.

```sh
kubectl version
```

**Flags**

| Flag | Effect |
|------|--------|
| `--client` | Only the local kubectl build (works offline) |
| `-o yaml` / `-o json` | Machine-readable output, good for scripts |
| `--short` | Compact one-liner per component (older versions) |

**Notes**

- If the cluster is unreachable you still get `Client Version`, but `Server Version` errors. That means the problem is connectivity or auth, not kubectl.
- Client and server should stay within one minor version of each other. A big skew is a common source of weird errors.
- In the course this was literally the "is the terminal connected to the cluster" test.
