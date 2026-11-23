# 10 · Declarative Manifests

The real way Kubernetes is used: write YAML, keep it in Git, `apply` it. Imperative commands are for learning and one-offs.

| Serial | Command |
|--------|---------|
| K8S-012 | `kubectl apply -f` |

## K8S-012 · `kubectl apply -f`

- **First seen:** [[Ch1 Pods]] → YAML Config (assignment)
- **Reused:** [[Ch2 Pod Management]] → Dry Run (dry-running an apply before doing it for real) · API Deployment (first deployment written from scratch) · [[Ch3 ConfigMaps]] → Config Maps · Applying the Config Map · Crawler · YAML Types · [[Ch4 Services]] → all three service manifests · [[Ch5 Namespaces]] → Creating a Namespace (re-applying with `metadata.namespace`) · [[Ch6 Resources]] → every limits/requests change · [[Ch7 Scaling]] → HPA manifests · [[Ch8 Health Checks]] → every probe change · [[Ch9 Local Cluster]] → Apply Everything (`apply -f .`) · [[Ch10 Gateway]] → Gateway install (`--server-side`, URL), routes, config deps · [[Ch12 Storage]] → config + deployment applies
- **Purpose:** Send a YAML file to the API server and let Kubernetes create or update the object to match it.

```sh
kubectl apply -f web-deployment.yaml
```

If the object already exists, `apply` patches it instead of erroring. Running it twice is safe and the second run reports `unchanged`.

**Flags**

| Flag | Effect |
|------|--------|
| `-f <file>` | One file, or a directory of manifests |
| `-f .` | Apply every manifest in the current directory |
| `-f a.yaml -f b.yaml` | Several files in one command |
| `-f <url>` | Apply a manifest straight from a URL |
| `--server-side` | Server-side apply: the API server computes the merge |
| `-f -` | Read manifest from stdin |
| `--dry-run=server` | Ask the API server what would change, change nothing |
| `--dry-run=client` | Validate locally and print what would be sent, no API call |
| `--prune` | Delete objects missing from the input (needs labels, use carefully) |
| `-k <dir>` | Apply a Kustomize directory |
| `-R` | Recurse into subdirectories |
| `--diff` | Show the diff instead of applying |

**The `last-applied-configuration` warning**

When you create an object imperatively (`kubectl create deployment ...`) and then `apply` a file for it, kubectl warns that the `kubectl.kubernetes.io/last-applied-configuration` annotation is missing. It is harmless: `apply` is recording the file's contents into that annotation so future applies can compute a proper three-way merge. The second apply is quiet.

**Dry run before applying**

```sh
kubectl apply -f web-deployment.yaml --dry-run=client
```

Validates the file and shows what would be done without touching the cluster. Use `--dry-run=server` when you need the API server's own validation, admission webhooks and defaulting included.

**Installing a controller from a URL**

```sh
kubectl apply --server-side -f https://github.com/envoyproxy/gateway/releases/download/v1.5.1/install.yaml
```

`-f` accepts a URL, so a controller install is one command. `--server-side` hands the merge to the API server instead of letting the client compute it, which is what large CRD bundles expect.

**Notes**

- `apply -f .` applies every YAML in a directory in one shot, which is how a repo of manifests rebuilds a whole app on a fresh cluster. Namespaces must exist first if the manifests reference them.
- `apply` is idempotent and declarative: it describes the desired state, the controller does the work. `create` is imperative and fails if the object exists.
- Keep manifests in Git. That is what makes Kubernetes infrastructure-as-code: review, history, rollback, one source of truth.
- Editing `kubectl edit` is a live patch with no record. The next `apply` of an older file silently reverts it. Edit the file, then apply.
- `--dry-run=client -o yaml` generates a manifest from an imperative command, the usual way to bootstrap a file.
