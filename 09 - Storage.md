# 09 · Storage

Keeping data alive when the pod is not. This category has **no command entries yet**: Chapter 12 configures storage entirely in YAML, and the commands it uses (`apply`, `logs`, `describe`, `get pods`, `get endpoints`) are already indexed elsewhere.

## Why the container filesystem is not enough

Anything written inside a container lives in its writable layer. It disappears when the pod is replaced, which happens on:

- `kubectl delete pod`
- a rolling update with a new image
- the node failing
- the pod being rescheduled onto another node

Saving to a file fixes *container* restarts but not *pod* replacement. The chapter demonstrates exactly that: messages survive a page refresh, then vanish after the pod is deleted.

## Volume types

| Type | Lifetime | Use for |
|------|----------|---------|
| `emptyDir` | Deleted with the pod | Scratch space, sharing files between containers in one pod |
| `hostPath` | Tied to one node's disk | Node agents only. Avoid for app data |
| `configMap` / `secret` | Managed by the object | Mounting config as files |
| `persistentVolumeClaim` | Survives pod deletion | Anything you actually want to keep |

**emptyDir** — three crawler containers writing to one directory:

```yaml
spec:
  volumes:
    - name: cache-volume
      emptyDir: {}
  containers:
    - name: synergychat-crawler-1
      volumeMounts:
        - name: cache-volume
          mountPath: /cache
```

Every container mounting `cache-volume` sees the same files. That is how the crawlers share one result set without a database.

## PV and PVC

| Object | Scope | Role |
|--------|-------|------|
| `PersistentVolume` (PV) | Cluster | The actual storage: an EBS volume, a GCE disk, an NFS share |
| `PersistentVolumeClaim` (PVC) | Namespace | A *request* for storage: size, access mode |
| `StorageClass` | Cluster | The provisioner that can create PVs on demand |

A pod never references a PV directly. It references a PVC, and Kubernetes binds that claim to a volume.

- **Static**: an admin creates PVs by hand, claims bind to matching ones.
- **Dynamic**: no matching PV exists, so the StorageClass provisions one for the claim. This is the normal cloud-native path.

**Reference PVC manifest** — the notes jump from the PV/PVC explanation straight to "attach persistence", so the claim itself is not shown. This is the standard shape:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: synergychat-api-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

**Attaching it** — same as any volume, with `claimName` instead of a volume type:

```yaml
spec:
  volumes:
    - name: synergychat-api-volume
      persistentVolumeClaim:
        claimName: synergychat-api-pvc
  containers:
    - name: synergychat-api
      volumeMounts:
        - name: synergychat-api-volume
          mountPath: /persist
```

Then point the app at the mounted path (`API_DB_FILEPATH: /persist/db.json`), not at an in-container path.

## Access modes

| Mode | Meaning |
|------|---------|
| `ReadWriteOnce` (RWO) | One **node** can mount read-write. Common default |
| `ReadOnlyMany` (ROX) | Many nodes read-only |
| `ReadWriteMany` (RWX) | Many nodes read-write, needs shared storage (NFS, CephFS), not typical block disks |

RWO is per node, not per pod, so several pods on the same node can share it. Multiple replicas spread across nodes generally cannot.

## Checking storage (not covered in the notes)

```sh
kubectl get pvc
kubectl get pv
kubectl describe pvc synergychat-api-pvc
```

`STATUS Bound` means the claim found a volume. `Pending` means nothing matched, often because no StorageClass exists or the requested size/mode is unavailable.

## Gotchas

- Deleting a **PVC** deletes the data, depending on the PV's reclaim policy (`Delete` by default for dynamic volumes). Deleting a **pod** does not.
- A Deployment's pods can all point at the same PVC only if they land on one node for RWO volumes. Do not assume shared storage across replicas.
- `emptyDir` survives container restarts inside a pod but not pod deletion. It also uses node disk, so a large one can fill the node.
- Resizing a PVC needs a StorageClass with `allowVolumeExpansion: true`. Shrinking is not supported.
- Mounting over an existing directory hides what was in the image at that path. Mount to a fresh path when in doubt.
- Multi-container pods share a **network namespace**, not just volumes: same IP, same ports. That is why the three crawler containers needed `8080`, `8081`, `8082`. See [[04 - Pods]] for the rest.

## Databases on Kubernetes

Running Postgres on a PVC works, but you own the hard parts: backups, version upgrades, failover, resource tuning. The usual rule of thumb is a managed service (Cloud SQL, RDS) for anything that matters, and in-cluster databases only for low-stakes data such as Prometheus or Grafana telemetry.
