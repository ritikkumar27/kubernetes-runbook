# 06 · Logs and Debugging

First stop when a pod is unhealthy. Everything here is read-only and safe to run against production.

| Serial | Command |
|--------|---------|
| K8S-006 | `kubectl logs` |

## K8S-006 · `kubectl logs`

- **First seen:** [[Ch1 Pods - 1]] → Ephemeral
- **Purpose:** Print what the container writes to stdout/stderr, the app's own view of what is happening.

```sh
kubectl logs PODNAME
```

**Flags**

| Flag | Effect |
|------|--------|
| `-f` | Follow, like `tail -f` |
| `--tail=100` | Last N lines only |
| `-c <container>` | Pick a container in a multi-container pod |
| `--previous` / `-p` | Logs from the **crashed previous** instance, the key flag for `CrashLoopBackOff` |
| `--since=10m` | Time-bounded output |
| `-l app=<name>` | Logs from every pod matching the label |

**Notes**

- Logs vanish with the pod. If you need crash output after a restart, `--previous` is the only way to see it, and only until the new container starts.
- `kubectl logs deployment/<name>` also works in modern kubectl and picks a pod for you.
- Nothing appearing? The app may log to a file instead of stdout, or the container never started (`ImagePullBackOff`, `Pending`). Then use `kubectl describe`.
- For a live shell instead of output, use `kubectl exec -it PODNAME -- sh`. Not covered in Ch1 yet.
