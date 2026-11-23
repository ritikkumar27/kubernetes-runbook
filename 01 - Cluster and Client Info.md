# 01 · Cluster and Client Info

Commands that tell you **what you're talking to** and **what it offers**.

| Serial | Command |
|--------|---------|
| K8S-001 | `kubectl version` |
| K8S-013 | `kubectl explain` |
| K8S-036 | `kubectl proxy` |

## K8S-001 · `kubectl version`

- **First seen:** [[Ch1 Pods]] → Welcome to "Learn Kubernetes"
- **Reused:** [[Ch2 Pod Management]] → The Control Plane (client vs server are separate programs, often on different machines) · [[Ch9 Local Cluster]] → `--client` to verify a fresh install without a cluster
- **Purpose:** Print the kubectl client version and the cluster's server version. First sanity check on any cluster.

```sh
kubectl version
```

**Flags**

| Flag | Effect |
|------|--------|
| `--client` | Only the local kubectl build (works offline) |
| `-o yaml` / `-o json` | Machine-readable output, good for scripts |
| `--short` | Compact one-liner per component (older versions) |

**Notes**

- If the cluster is unreachable you still get `Client Version`, but `Server Version` errors. That means the problem is connectivity or auth, not kubectl.
- Client and server should stay within one minor version of each other. A big skew is a common source of weird errors.
- In the course this was literally the "is the terminal connected to the cluster" test.
- The two halves are different programs: `kubectl` on your laptop, `kube-apiserver` (usually a pod in `kube-system`) in the cluster.

## K8S-013 · `kubectl explain`

- **First seen:** [[Ch2 Pod Management]] → Kubectl Explain
- **Purpose:** Print Kubernetes' own documentation for any field, straight in the terminal. The offline answer to "what does this key do?"

```sh
kubectl explain deployment.spec.replicas
```

The argument is a dot-separated path through the YAML.

**Flags**

| Flag | Effect |
|------|--------|
| `--recursive` | Show the whole subtree, every nested field |
| `--api-version=apps/v1` | Pick the API version when a kind exists in several |
| `--output=plaintext` | Default human output. There is also `plaintext-openapiv2` |

**Notes**

- The `FIELDS` section at the bottom lists the keys you are allowed to nest under that path. That is the fastest way to discover YAML you did not know existed.
- It stops at keys **you** define. `deployment.spec.selector` explains `matchLabels`, but `deployment.spec.selector.matchLabels.app` fails because `app` is your label, not a schema field.
- `kubectl explain pods --recursive` is enormous but occasionally the fastest lookup. Pipe it to `less`.
- Works on any resource the cluster's API server knows about, so it also tells you what your specific cluster version supports.


## K8S-036 · `kubectl proxy`

- **First seen:** [[Ch9 Local Cluster]] → Kubectl Proxy
- **Purpose:** Run a local HTTP proxy that forwards requests to the API server with your kubeconfig credentials attached, so you can hit the raw API from a browser or `curl`.

```sh
kubectl proxy
```

Then open `http://127.0.0.1:8001/api/v1/namespaces/default/pods`.

**Flags**

| Flag | Effect |
|------|--------|
| `--port=8001` | Listen port (default 8001) |
| `--address=127.0.0.1` | Bind address. Keep it on localhost |
| `--accept-hosts='^localhost$,^127\.0\.0\.1$'` | Host header allowlist |
| `--api-prefix=/` | Prefix for API paths |

**Notes**

- It **blocks** and runs until `Ctrl+C`, so give it its own terminal. Tools that inspect your cluster locally rely on it staying up.
- Authentication is handled for you: requests arrive on plain HTTP locally and leave signed with your kubeconfig credentials.
- Useful paths: `/api/v1/namespaces/<ns>/pods`, `/apis/apps/v1/deployments`, and `/openapi/v2` for the full schema.
- `jq` turns that "big nasty JSON blob" into something readable: `curl -s localhost:8001/api/v1/namespaces/default/pods | jq '.items[].metadata.name'`.
- Security: it grants full access to whatever your kubeconfig can do, with no further auth. Never bind it to `0.0.0.0` on a shared machine.
- `kubectl proxy` is a debugging and learning tool, not how production traffic reaches workloads. That is a Service, Ingress or Gateway.
