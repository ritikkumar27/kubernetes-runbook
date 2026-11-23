# 14 · Local Cluster and Tooling

Setting up a cluster on your own machine, and the non-`kubectl` commands that come with it.

| Serial | Command |
|--------|---------|
| K8S-034 | `minikube addons enable` |

## Installing kubectl

```sh
# Linux / WSL2, x86-64
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Linux / WSL2, ARM64
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/arm64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

Verify with `kubectl version --client`. On macOS use the official instructions or Homebrew; the point is just to get the binary on `PATH`.

**Editor for `kubectl edit`**

```sh
# in ~/.zshrc or ~/.bashrc
export EDITOR="code -w"   # VS Code
export EDITOR="zed -w"    # Zed
```

The `-w` (wait) flag is required: without it the editor returns immediately and `kubectl edit` applies an empty or unchanged buffer. `KUBE_EDITOR` takes precedence over `EDITOR`, and `vi` is the fallback.

## K8S-034 · `minikube addons enable`

- **First seen:** [[Ch9 Local Cluster]] → Metrics
- **Purpose:** Turn on a Minikube addon. Here it installs metrics-server, without which `kubectl top` cannot report anything.

```sh
minikube addons enable metrics-server
```

**Related**

| Command | Effect |
|---------|--------|
| `minikube addons list` | Show every addon and whether it is enabled |
| `minikube addons disable <name>` | Turn one off |
| `minikube start` | Boot the single-node cluster |
| `minikube dashboard` | Open the web UI |

**Notes**

- This is a **minikube** command, not kubectl. It changes your local cluster only, so it never appears in a manifest or in Git.
- After enabling, `kube-system` gains a `metrics-server` pod. Give it a couple of minutes: `kubectl -n kube-system get pod` until it is `1/1`.
- A managed cluster already ships metrics-server. On a bare cluster you would install it from the metrics-server manifests instead.
- Metrics are what feed `kubectl top` **and** the HorizontalPodAutoscaler, so a missing metrics-server silently disables both.

## Local vs production

| | Minikube (default) | Production |
|---|---|---|
| Nodes | 1 | Many, often hundreds or thousands |
| Control plane | Local container | Managed (GKE/EKS/AKS) or self-run |
| Failure tolerance | None, one machine | Survives node loss |
| Cost | Free | Real money |
| Purpose | Learning, local testing | Serving users |

A single node means one resource ceiling. Once the machine is full, extra pods stay `Pending` with `Insufficient cpu` or `Insufficient memory` — there is no second node for the scheduler to try. That is the practical difference you feel while learning.
