# 11 · Rollouts and Updates

Shipping a new version without downtime, and getting back when it goes wrong.

| Serial | Command |
|--------|---------|
| K8S-014 | `kubectl rollout status deployment` |
| K8S-015 | `kubectl rollout history deployment` |
| K8S-016 | `kubectl rollout restart deployment` |
| K8S-017 | `kubectl rollout undo deployment` |

## How a rollout works

Changing anything under `spec.template` (image, env, resources) starts a rolling update:

1. A **new ReplicaSet** is created for the new pod template, with a new `pod-template-hash`.
2. The new ReplicaSet scales **up** a pod or two at a time.
3. The old ReplicaSet scales **down** as new pods become ready.

The old ReplicaSet is kept at `0` replicas, which is what makes rollbacks instant. `spec.revisionHistoryLimit` (default `10`) controls how many are retained; older ones are garbage collected.

## K8S-014 · `kubectl rollout status deployment`

- **First seen:** [[Ch2 Pod Management]] → Rollouts (assignment)
- **Purpose:** Block and watch a rollout until it finishes or fails.

```sh
kubectl rollout status deployment synergychat-web
```

**Flags**

| Flag | Effect |
|------|--------|
| `--timeout=120s` | Give up after a duration |
| `-w=true` | Watch (default). `-w=false` prints current state once |
| `-n <namespace>` | Target namespace |

**Notes**

- Output looks like `Waiting for deployment "synergychat-web" rollout to finish: 1 out of 3 new replicas have been updated...`.
- Exits non-zero on failure, so it is safe to use in CI: `kubectl rollout status ... && echo shipped`.
- A stuck rollout usually means new pods are not becoming Ready. `kubectl get pods` and `describe` point at the cause.

## K8S-015 · `kubectl rollout history deployment`

- **First seen:** [[Ch2 Pod Management]] → Rollouts (assignment)
- **Purpose:** List the numbered revisions of a Deployment's pod template.

```sh
kubectl rollout history deployment synergychat-web
```

**Flags**

| Flag | Effect |
|------|--------|
| `--revision=N` | Show the details of one revision |
| `-n <namespace>` | Target namespace |

**Notes**

- Revisions map to ReplicaSets. Every template change (including `rollout restart`) makes a new one.
- The `CHANGE-CAUSE` column is empty unless someone sets the `kubernetes.io/change-cause` annotation (the old `--record` flag did this and is deprecated).
- Only the last `revisionHistoryLimit` revisions (default 10) are kept, so old numbers eventually disappear.

## K8S-016 · `kubectl rollout restart deployment`

- **First seen:** [[Ch2 Pod Management]] → Rollouts
- **Purpose:** Replace every pod with a fresh one without changing the image or config.

```sh
kubectl rollout restart deployment synergychat-web
```

**Notes**

- Under the hood it stamps a `kubectl.kubernetes.io/restartedAt` annotation on the pod template, which is a template change, so a new ReplicaSet and a normal rolling update follow.
- This is the "turn it off and on again" of Kubernetes: use it to pick up a changed ConfigMap/Secret, clear a wedged pod, or force a fresh image pull.
- Zero downtime, because it obeys the same surge/unavailable rules as any rollout.
- It is a cluster-side change only. Your YAML file is untouched, so it does not show up in Git.

## K8S-017 · `kubectl rollout undo deployment`

- **First seen:** [[Ch2 Pod Management]] → Rollbacks
- **Purpose:** Roll back to the previous revision by scaling the old ReplicaSet back up.

```sh
kubectl rollout undo deployment synergychat-web
```

**Flags**

| Flag | Effect |
|------|--------|
| `--to-revision=N` | Roll back to a specific revision instead of the previous one |
| `-n <namespace>` | Target namespace |

**Going further back**

```sh
kubectl rollout history deployment synergychat-web
kubectl rollout undo deployment synergychat-web --to-revision=2
```

**Notes**

- It is a normal rolling update in reverse: old ReplicaSet up, current one down. Users should not notice.
- The rolled-back template becomes the newest revision number, so "undo the undo" is just another undo.
- **Cluster-only.** The bad config is still in your repo, so the next `kubectl apply -f` will break it again. Fix the file too.
- If the bad deploy never became Ready, the old pods are still serving traffic, which is exactly why the rollout is safe.

## Rollout-related pod states

Worth knowing when the rollout is stuck:

| Status | Meaning |
|--------|---------|
| `ErrImagePull` | The last image pull attempt failed |
| `ImagePullBackOff` | Pull failed and Kubernetes is backing off, waiting longer each retry |
| `CrashLoopBackOff` | Container starts, exits, restarts, repeats |

Usual causes: typo in the image name, a tag that does not exist, or a private registry with no credentials. `kubectl describe pod` shows the `Failed` events with the real reason.
