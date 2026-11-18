# 04 · Pods

Pod-level lifecycle actions. You rarely create bare Pods in real work, but you delete, inspect and troubleshoot them constantly.

| Serial | Command |
|--------|---------|
| K8S-007 | `kubectl delete pod` |

## K8S-007 · `kubectl delete pod`

- **First seen:** [[Ch1 Pods - 1]] → Ephemeral
- **Reused:** Ch1 · Unique IP Addresses (deleting a pod to watch its IP change)
- **Purpose:** Terminate a Pod. Demonstrates that Pods are disposable and that a Deployment replaces them.

```sh
kubectl delete pod PODNAME
```

**Flags**

| Flag | Effect |
|------|--------|
| `--grace-period=<seconds>` | Shutdown window (default 30) |
| `--force --grace-period=0` | Kill immediately, only when a pod is stuck `Terminating` |
| `--wait=false` | Return before deletion finishes |

**Notes**

- Deleting a **Pod** is not "undoing a deployment". The Deployment still wants N replicas, so a replacement appears almost instantly.
- To really reduce the count, change the Deployment with `scale` or edit `replicas`. Do not delete pods one by one.
- Deleting a Pod that is **not** owned by a controller removes it permanently.
- Pods get SIGTERM, then the grace period, then SIGKILL, so well-behaved apps can drain.
