# 05 · Scaling

Changing the number of running copies. Cheap to do, easy to forget to undo.

| Serial | Command |
|--------|---------|
| K8S-005 | `kubectl scale deployment` |

## K8S-005 · `kubectl scale deployment`

- **First seen:** [[Ch1 Pods - 1]] → What Is Kubernetes
- **Reused:** [[Ch2 Pod Management]] → Scaling (the walkthrough of what the cluster does after `scale`)
- **Purpose:** Change a Deployment's replica count in one shot, without an editor.

```sh
kubectl scale deployment synergychat-web --replicas=20
```

Verify with `kubectl get pods`, you should see 20 pods in `Running`.

**Flags**

| Flag | Effect |
|------|--------|
| `--replicas=N` | Target count (required). `0` scales to zero |
| `--current-replicas=N` | Safety guard: only scale if the current count matches, so you do not clobber a concurrent change |
| `--timeout=<duration>` | How long to wait for the operation |

**Notes**

- Scaling writes `spec.replicas` on the Deployment. The ReplicaSet then creates or deletes Pods to match.
- Scaling down is graceful, pods terminate in order rather than all at once.
- Same "live edit, not in Git" caveat as `edit`: a later `apply` of the old manifest reverts the count.
- `kubectl scale` is the manual version of the HorizontalPodAutoscaler (`kubectl autoscale`), which is what production usually runs.
