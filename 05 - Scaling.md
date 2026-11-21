# 05 · Scaling

Changing the number of running copies. Cheap to do, easy to forget to undo.

| Serial | Command |
|--------|---------|
| K8S-005 | `kubectl scale deployment` |
| K8S-031 | `kubectl get hpa` |
| K8S-032 | `kubectl describe hpa` |

## K8S-005 · `kubectl scale deployment`

- **First seen:** [[Ch1 Pods]] → What Is Kubernetes
- **Reused:** [[Ch2 Pod Management]] → Scaling (the walkthrough of what the cluster does after `scale`) · [[Ch7 Scaling]] → Scaling Down (manual bump while an HPA is active)
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

## Vertical vs horizontal

| | Vertical (scale up) | Horizontal (scale out) |
|---|---|---|
| What changes | More CPU/RAM per pod | More pods |
| Ceiling | The largest node you can buy | Number of nodes you can add |
| Downtime risk | Often needs a restart | None, pods roll |
| Kubernetes default choice | Rare | Preferred |

Kubernetes is built for horizontal scaling, so autoscaling means adding pods, not bigger pods.

## K8S-031 · `kubectl get hpa`

- **First seen:** [[Ch7 Scaling]] → Horizontal Pod Autoscaling (assignment)
- **Purpose:** See the autoscaler's current target, its min/max bounds, and the live replica count.

```sh
kubectl get hpa -n crawler
```

Typical columns: `NAME  REFERENCE  TARGETS  MINPODS  MAXPODS  REPLICAS  AGE`.

`TARGETS  45%/50%` means the average CPU utilization is 45% against a 50% target. `REPLICAS` is what the HPA currently wants.

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Target namespace |
| `-A` | All namespaces |
| `-o yaml` | Full object, including last scale event |
| `-w` | Watch it react to load |

**Notes**

- `hpa` is the short form; an HPA is just another resource, so `get` works like anywhere else.
- `TARGETS  <unknown>/50%` means metrics are not available yet: metrics-server missing, still warming up, or the pod has no CPU request to measure against. This is the most common "why isn't my autoscaler doing anything".
- The HPA only targets a scalable resource (Deployment, ReplicaSet, StatefulSet) through `scaleTargetRef`. It never touches pods directly.
- Do not set `replicas` in the Deployment manifest you are autoscaling. The HPA owns that field, and the two will fight.

## K8S-032 · `kubectl describe hpa`

- **First seen:** [[Ch7 Scaling]] → Scaling Down (finding `SuccessfulRescale`)
- **Purpose:** Read the autoscaler's metrics, thresholds and the event log explaining every scale decision.

```sh
kubectl describe hpa web-hpa
```

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Target namespace |
| `-l app=<name>` | Describe matching HPAs |

**Notes**

- `Metrics:` shows the current value, the target, and the formula result.
- `Events:` is the useful part. `SuccessfulRescale` records old → new replica count and the reason ("cpu resource utilization (percentage of request) above target").
- `FailedGetResourceMetric` or `missing request for cpu` means it cannot compute utilization at all, so it will not scale.
- Scale-up is fast. Scale-down is deliberately slow: a stabilization window (5 minutes by default) prevents flapping while traffic dips.

## How the HPA decides

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: crawler-hpa
  namespace: crawler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: synergychat-crawler
  minReplicas: 1
  maxReplicas: 4
  targetCPUUtilizationPercentage: 50
```

Simplified algorithm:

```text
desiredReplicas = ceil(currentReplicas * currentUtilization / targetUtilization)
```

then clamped between `minReplicas` and `maxReplicas`.

**The percentage is of the CPU *request*, not the limit.**

- Limit set, no request: the request defaults to the limit, so utilization still works.
- Neither set: utilization is undefined, and the HPA cannot scale. Always set CPU requests on autoscaled workloads.
- `autoscaling/v1` only supports CPU. `autoscaling/v2` adds memory and custom/external metrics, and is what you would write today.

**Scale to zero** is a different feature (KEDA or an HPA `minReplicas: 0` with a compatible setup). A plain HPA keeps at least one pod.
