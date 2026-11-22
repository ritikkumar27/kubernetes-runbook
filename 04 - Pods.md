# 04 · Pods

Pod-level lifecycle actions. You rarely create bare Pods in real work, but you delete, inspect and troubleshoot them constantly.

| Serial | Command |
|--------|---------|
| K8S-007 | `kubectl delete pod` |

## K8S-007 · `kubectl delete pod`

- **First seen:** [[Ch1 Pods]] → Ephemeral
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

## Health checks (probes)

Probes live in the container spec, but they decide a pod's `READY` and `RESTARTS` columns, so they belong here. They are YAML, not commands, which is why Chapter 8 adds no new serials.

| Probe | Question it answers | On failure |
|-------|--------------------|------------|
| `startupProbe` | Has it finished booting? | Restart the container |
| `livenessProbe` | Is it still alive and responding? | Restart the container |
| `readinessProbe` | Should it receive traffic **right now**? | Remove from Service endpoints, **no restart** |

A failed readiness probe leaves the pod `Running` with `0/1` ready and `RESTARTS 0`. That combination is the signature of a broken readiness probe, not a crash.

**Shared fields**

| Field | Default | Meaning |
|-------|---------|---------|
| `initialDelaySeconds` | 0 | Wait before the first check |
| `periodSeconds` | 10 | Time between checks |
| `timeoutSeconds` | 1 | Per-check timeout |
| `failureThreshold` | 3 | Consecutive failures before acting |
| `successThreshold` | 1 | Consecutive successes to be considered healthy |

An `httpGet` probe treats **200–399** as success. `path` + `port` are the usual pair; `exec` and `tcpSocket` are the alternatives.

**Startup probe ordering**

Until a `startupProbe` succeeds, liveness and readiness are **not run**. That fixes slow-booting apps: give the startup probe a generous budget (`failureThreshold × periodSeconds`), then keep the liveness probe aggressive so a hung app is replaced quickly.

```yaml
startupProbe:
  httpGet: { path: /healthz, port: 8080 }
  periodSeconds: 2
  failureThreshold: 30     # up to 60s to boot
livenessProbe:
  httpGet: { path: /healthz, port: 8080 }
  periodSeconds: 5
  failureThreshold: 3      # dead within ~15s after boot
```

**Notes**

- Readiness failures give you zero-downtime maintenance windows: return "not ready" while rebuilding a cache, and the Service simply routes elsewhere.
- A rolling update will **not** remove the old pod until the new one is ready. A permanently unready new pod therefore leaves the old one serving, which is why a broken probe shows `2` pods instead of a full outage.
- Reverting the probe to the previous template makes Kubernetes keep the existing pod and scale the broken ReplicaSet to `0`. It does not always trigger a fresh rollout.
- `kubectl describe pod` prints `Startup:`, `Liveness:` and `Readiness:` blocks, so you can confirm what is configured without opening the YAML.
