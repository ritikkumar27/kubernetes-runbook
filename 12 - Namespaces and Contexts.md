# 12 · Namespaces and Contexts

Grouping resources, and not typing `-n` on every command.

| Serial | Command |
|--------|---------|
| K8S-023 | `kubectl create namespace` |
| K8S-024 | `kubectl get namespaces` |
| K8S-028 | `kubectl config get-contexts` |
| K8S-029 | `kubectl config set-context --current` |

## How namespaces work

- A namespace is a folder for Kubernetes objects. It isolates names, not security (that is RBAC's job).
- Uniqueness is **(kind, namespace, name)**. Two Deployments can both be `api`, as long as they are in different namespaces.
- That is also how `kubectl apply` decides create vs update: same kind + name + namespace = update.
- `default` exists automatically and is used when you do not pass `-n`.
- `kube-system` holds the control plane components (API server, etcd, scheduler, controller manager, kube-proxy, CoreDNS). Do not touch it.

## K8S-023 · `kubectl create namespace`

- **First seen:** [[Ch5 Namespaces]] → Creating a Namespace (assignment)
- **Reused:** [[Ch9 Local Cluster]] → Apply Everything (recreating `crawler` on a fresh context)
- **Purpose:** Create a namespace.

```sh
kubectl create ns crawler
```

**Flags**

| Flag | Effect |
|------|--------|
| `--dry-run=client -o yaml` | Print the Namespace manifest instead of creating it |

**Notes**

- `ns` is the short form of `namespace`.
- The YAML equivalent is `apiVersion: v1`, `kind: Namespace`, `metadata.name`.
- Deleting a namespace deletes **everything inside it**. `kubectl delete ns crawler` is a very sharp tool.
- Because moving a resource to another namespace creates a *new* object, `apply` reports `created`, not `configured`. The old copy stays behind until you delete it.

## K8S-024 · `kubectl get namespaces`

- **First seen:** [[Ch5 Namespaces]] → Creating a Namespace (assignment)
- **Purpose:** List namespaces and confirm yours exists.

```sh
kubectl get ns
```

Typical columns: `NAME  STATUS  AGE`. `STATUS` is `Active`, or `Terminating` while deletion finishes.

**Flags**

| Flag | Effect |
|------|--------|
| `-o wide` | Adds labels |
| `--show-labels` | Label column |
| `-o yaml` | Full object |

**Notes**

- You will almost always see `default`, `kube-system`, `kube-public` and `kube-node-lease`.
- `get ns` is one of the few commands where `-n` is irrelevant: namespaces are cluster-scoped.
- In a real cluster, `kubectl get pods -A` plus `--field-selector` is usually faster than walking namespaces one at a time.

## K8S-028 · `kubectl config get-contexts`

- **First seen:** [[Ch5 Namespaces]] → Contexts (assignment)
- **Purpose:** Show the contexts in your kubeconfig and which one is active.

```sh
kubectl config get-contexts
```

Typical columns: `CURRENT  NAME  CLUSTER  AUTHINFO  NAMESPACE`. The `*` marks the active context.

**Flags**

| Flag | Effect |
|------|--------|
| `-o name` | Machine-readable list |
| `--kubeconfig=<path>` | Use a different kubeconfig file |

**Notes**

- A context is a named triple: **cluster** (which API server), **user** (which credentials), **namespace** (default `-n`).
- The file lives at `~/.kube/config` unless `$KUBECONFIG` says otherwise. Contexts are how one laptop talks to dev, staging and prod.
- `kubectl config current-context` prints just the active name, handy in a prompt or script.
- `kubectl config use-context <name>` switches cluster/user. `set-context --current` modifies the one you are on.

## K8S-029 · `kubectl config set-context --current`

- **First seen:** [[Ch5 Namespaces]] → Contexts (assignment)
- **Purpose:** Change the default namespace of the active context, so plain `kubectl` commands target it.

```sh
kubectl config set-context --current --namespace=crawler
```

After that, `kubectl get pods` shows only the `crawler` namespace.

**Flags**

| Flag | Effect |
|------|--------|
| `--current` | Apply to the active context |
| `--namespace=<ns>` | The new default namespace |
| `--cluster=<name>` | Point the context at another cluster |
| `--user=<name>` | Change the credentials used |

**Notes**

- `--current` is shorthand for naming the context explicitly: `kubectl config set-context my-context --namespace=crawler`.
- It writes to the kubeconfig file, so the change survives restarts of your shell.
- **Check the context before deleting anything.** The default namespace silently decides what `kubectl delete` hits, and a "harmless" command aimed at staging can land on prod.
- `kubectl config view` prints the whole merged config when something is confusing. `minify=true` hides the repetitive parts.
- To undo, set the namespace back to `default`.
