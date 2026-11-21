# 03 · Workloads and Deployments

Create and change the thing that **owns** your pods. Day to day you touch Deployments far more than raw Pods.

| Serial | Command |
|--------|---------|
| K8S-002 | `kubectl create deployment` |
| K8S-008 | `kubectl edit deployment` |
| K8S-025 | `kubectl delete deployment` |

## K8S-002 · `kubectl create deployment`

- **First seen:** [[Ch1 Pods]] → Deployments
- **Reused:** Ch1 · Deployment and Pod Review · YAML Config (the "quick and dirty" creation that causes the `last-applied-configuration` warning) · [[Ch2 Pod Management]] → Dry Run (generating a clean manifest)
- **Purpose:** Imperatively create a Deployment from a container image. Kubernetes then creates the ReplicaSet and Pods for you.

```sh
kubectl create deployment synergychat-web --image=bootdotdev/synergychat-web:latest
```

Two inputs are mandatory: the Deployment **name** and the **image ID**.

**Flags**

| Flag | Effect |
|------|--------|
| `--image=<repo:tag>` | Image to run (required) |
| `--replicas=N` | Start with N pods (default 1) |
| `--port=N` | Declare the container port |
| `--dry-run=client -o yaml` | Print the manifest instead of creating it |

**Generating a clean manifest**

```sh
kubectl create deployment synergychat-web --image=bootdotdev/synergychat-web:latest --replicas=3 --dry-run=client -o yaml > web-deployment.yaml
```

`--dry-run=client` means "build the object, do not send it". Combined with `-o yaml` you get a minimal manifest: no `status`, no `uid`, no `resourceVersion`, no timestamps. This is the right file to commit, unlike a `get -o yaml` dump, which is noisy and can even cause a `Conflict` error if a stale `resourceVersion` is applied.

**Notes**

- This is the *imperative* style. Real projects keep a YAML file and run `kubectl apply -f`. Use `--dry-run=client -o yaml` to generate that starting point.
- Creating a Deployment declares the desired state. The Deployment controller then loops: current state becomes desired state.
- Never edit the ReplicaSet or Pods the Deployment owns, they get recreated. Change the Deployment instead.
- Pinning `:latest` is fine for a course but bad in production: rollbacks become ambiguous.

## K8S-008 · `kubectl edit deployment`

- **First seen:** [[Ch1 Pods]] → Desired State
- **Reused:** Ch1 · YAML Config (contrasted with the declarative `apply -f` workflow)
- **Purpose:** Open the live object as YAML in your editor and apply the change on save. Handy for a quick fix in a lab.

```sh
kubectl edit deployment synergychat-web
```

Then change, for example:

```yaml
spec:
  replicas: 2
```

**Notes**

- Editor order: `$KUBE_EDITOR`, then `$EDITOR`, then `vi`.
- Saving sends a patch to the API server. The controller reconciles immediately, so pods appear or disappear right away.
- Closing the editor without saving changes nothing.
- It is a live, imperative edit and nothing records it in Git. A later `apply` of the old manifest will revert you. For real work, edit the YAML file under version control and `kubectl apply` it.
- To skip the editor entirely, use `kubectl patch deployment <name> -p '{"spec":{"replicas":2}}'`.

## K8S-025 · `kubectl delete deployment`

- **First seen:** [[Ch5 Namespaces]] → Creating a Namespace (cleaning up the old `default` namespace copies)
- **Purpose:** Remove a Deployment and, by default, everything it owns.

```sh
kubectl delete deployment synergychat-crawler
```

**Flags**

| Flag | Effect |
|------|--------|
| `--cascade=background` | Default: delete the Deployment, its ReplicaSet and its Pods |
| `--cascade=orphan` | Delete only the Deployment, leave the Pods running |
| `--wait=false` | Do not block until the objects are gone |
| `-n <namespace>` | Target namespace |

**Notes**

- The cascade is the point: deleting a Deployment is the real "stop this app", unlike deleting a Pod, which gets replaced.
- Like `delete pod`, this is a cluster action. The manifest in Git is untouched, so a later `apply -f` brings it right back.
- Deleting the whole namespace does the same thing to every object inside it, all at once.
- Double-check the active context first if you switched namespaces with `config set-context`.
