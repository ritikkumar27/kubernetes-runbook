# 02 · Viewing and Inspecting Resources

The commands you type most often. Read-only, safe, and the basis of every debugging session.

| Serial | Command |
|--------|---------|
| K8S-003 | `kubectl get deployments` |
| K8S-004 | `kubectl get pods` |
| K8S-009 | `kubectl describe pod` |
| K8S-010 | `kubectl describe deployment` |
| K8S-011 | `kubectl get replicasets` |

## K8S-003 · `kubectl get deployments`

- **First seen:** [[Ch1 Pods]] → Deployments (verifying the create)
- **Reused:** Ch1 · YAML Config (saving the current configuration with `-o yaml`) · [[Ch2 Pod Management]] → Dry Run (comparing the noisy live YAML with a clean generated manifest)
- **Purpose:** List Deployments in the current namespace and see whether they are healthy.

```sh
kubectl get deployments
```

Typical columns: `NAME  READY  UP-TO-DATE  AVAILABLE  AGE`. A `READY 1/1` means 1 desired replica, 1 actually available.

**Flags**

| Flag | Effect |
|------|--------|
| `-o wide` | Adds selector and image columns |
| `-o yaml` | Full object, same shape as the manifest |
| `-n <namespace>` | Target another namespace |
| `-A` | All namespaces |
| `-w` | Stream updates live (`Ctrl+C` to stop) |
| `--show-labels` | Show label columns |

**Exporting a live object to a file**

```sh
kubectl get deployment synergychat-web -o yaml > my-deployment.yaml
```

`-o yaml` dumps the object as YAML and the shell redirect writes it to disk. That file is a valid manifest you can edit and later `kubectl apply -f`.

**Notes**

- `deploy` is the accepted short form: `kubectl get deploy`.
- READY is the only column that matters at a glance. `0/1` or `0/20` means the controller cannot satisfy the desired state.
- A YAML dump includes `status` and `metadata` filled in by the cluster. When you apply it back, Kubernetes ignores `status`, so it is safe to leave in.
- The old `--export` flag is deprecated. Just use `-o yaml`.

## K8S-004 · `kubectl get pods`

- **First seen:** [[Ch1 Pods]] → Pods
- **Reused:** Ch1 · What Is Kubernetes (after scaling) · Ephemeral · Unique IP Addresses (`-o wide`) · [[Ch2 Pod Management]] → The Control Plane (`-n kube-system`) · Labels (`-l`, `--show-labels`) · Rollbacks · Thrashing Pods (watching RESTARTS climb) · [[Ch3 ConfigMaps]] → Config Maps · Config Errors · Crawler
- **Purpose:** List Pods and their status. The bread-and-butter health check.

```sh
kubectl get pods
```

Typical columns: `NAME  READY  STATUS  RESTARTS  AGE`.

**Flags**

| Flag | Effect |
|------|--------|
| `-o wide` | Adds Pod IP and node name |
| `-A` | All namespaces (essential when you cannot find "missing" pods) |
| `-w` | Watch pods appear and disappear, great for observing self-healing |
| `-l app=<name>` | Filter by label |
| `--show-labels` | Add a LABELS column |
| `--field-selector status.phase=Running` | Filter by field |

**Unique IPs**

```sh
kubectl get pods -o wide
```

Every pod gets its own cluster-internal IP, a different one from the node it runs on. Delete a pod and its replacement gets a fresh address, sometimes the same one recycled. That is exactly why apps should not talk to pod IPs directly, and why Services exist (later chapter).

**Filtering by label**

```sh
kubectl get pods -l app=synergychat-web
kubectl get pods --show-labels
```

`-l` keeps only objects matching the label selector. `--show-labels` adds a LABELS column so you can see `app=synergychat-web` and the `pod-template-hash` the ReplicaSet adds.

Labels, not names, are how ownership works: a Deployment continuously counts pods that match its selector, so a replaced pod with a brand new name is still counted.

**Notes**

- Pod names look like `synergychat-web-xxxxyyyzzz-xxyyz`: `<deployment>-<replicaset-hash>-<random>`. The hash names the ReplicaSet that owns it.
- STATUS values you should read as a story: `ContainerCreating`, `Running`, `Completed`, `CrashLoopBackOff`, `ImagePullBackOff`, `Pending`, `Terminating`.
- RESTARTS climbing means the container keeps crashing. Jump to `kubectl logs` or `describe` next.
- A Pod deleted by hand under a Deployment is replaced automatically. The desired state always wins.

## K8S-009 · `kubectl describe pod`

- **First seen:** [[Ch1 Pods]] → Describe
- **Reused:** [[Ch2 Pod Management]] → Image Pull Errors (events name the real pull failure) · [[Ch3 ConfigMaps]] → Config Errors (events name the missing key)
- **Purpose:** Print a long human-readable report about one pod, including the events timeline that `get` never shows.

```sh
kubectl describe pod POD_NAME
```

**What the report contains**

| Section | Why you care |
|---------|--------------|
| Labels / IP / Node | Where the pod actually landed |
| Containers | Image, state, restart count |
| Conditions | `Ready`, `ContainersReady`, `PodScheduled` |
| Volumes | Mounts and their sources |
| Events | Everything Kubernetes did, oldest first |

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Describe in another namespace |
| `-l app=<name>` | Describe every pod matching a label |
| `--show-events=false` | Hide the events (rarely useful) |

**Notes**

- Events sit at the bottom and are usually the most valuable part: scheduled → image pulled → container created → started.
- For a broken pod, read events top to bottom. `FailedScheduling`, `Failed to pull image`, `Back-off restarting` each point at a different fix.
- Events expire (default ~1 hour), so look early. Cluster-wide view: `kubectl get events --sort-by=.lastTimestamp`.
- `describe` works on most object kinds, not only pods.

## K8S-010 · `kubectl describe deployment`

- **First seen:** [[Ch1 Pods]] → Describe (assignment)
- **Purpose:** See the deployment's replicas, strategy, selector, pod template and rollout events in one page.

```sh
kubectl describe deployment synergychat-web
```

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Target another namespace |
| `-l app=<name>` | Describe matching deployments |

**Notes**

- Check `Replicas: 3 desired | 3 updated | 3 total | 3 available` to see the gap between desired and reality.
- `StrategyType: RollingUpdate` plus `Max Unavailable` / `Max Surge` explains how updates replace pods.
- `Conditions` going `Progressing=False` with reason `ProgressDeadlineExceeded` is the classic "image never became available" signal.
- `Events` here are deployment/ReplicaSet level, not container level. For container detail go down to the pod.

## K8S-011 · `kubectl get replicasets`

- **First seen:** [[Ch1 Pods]] → Replica Sets
- **Reused:** [[Ch2 Pod Management]] → Rollouts (new ReplicaSet at 3 pods, old one scaled to 0)
- **Purpose:** See the ReplicaSets that Deployments created, and which pod-template hash each one runs.

```sh
kubectl get replicasets
```

Typical columns: `NAME  DESIRED  CURRENT  READY  AGE`.

**Flags**

| Flag | Effect |
|------|--------|
| `-o wide` | Adds the image each ReplicaSet runs |
| `-o yaml` | Full spec, including the `pod-template-hash` label |
| `-A` | All namespaces |

**Notes**

- You never create these by hand. The Deployment does it. A Deployment is a wrapper around a ReplicaSet.
- The ReplicaSet name is `<deployment>-<hash>`, and that same hash appears in pod names, which is how you map pods back to a revision.
- After a new rollout, the old ReplicaSet stays around with `DESIRED 0` so you can roll back. That is `revisionHistoryLimit`, default 10.
- Do not delete them manually. You would fight the Deployment controller.
