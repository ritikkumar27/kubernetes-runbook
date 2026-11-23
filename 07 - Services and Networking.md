# 07 · Services and Networking

Pods come and go with new IPs. A Service is the stable address in front of them.

| Serial | Command |
|--------|---------|
| K8S-021 | `kubectl get endpoints` |
| K8S-022 | `kubectl get services` |
| K8S-026 | `kubectl delete service` |
| K8S-038 | `kubectl get gateway` |
| K8S-039 | `kubectl describe httproute` |

## How a Service works

- It has a **label selector**, not a list of pods. Every pod matching the selector joins the pool automatically.
- It gets its own stable **ClusterIP**, with a DNS name `service-name.namespace.svc.cluster.local`.
- Clients talk to the Service, never to pod IPs, so pod replacement is invisible to them.
- Traffic is load-balanced across the healthy pods in the pool (via `kube-proxy` rules on each node).
- The Service's **endpoints** are the concrete `podIP:port` pairs it currently routes to.

**Anatomy**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: synergychat-web
  ports:
    - protocol: TCP
      port: 80          # what the Service listens on
      targetPort: 8080  # what the container listens on
```

**Service types**

| Type | What it adds | Use when |
|------|--------------|----------|
| `ClusterIP` (default) | Internal-only virtual IP | Service-to-service inside the cluster |
| `NodePort` | A static port on every node's IP (`nodePort: 30080`) | Quick external access, labs, on-prem |
| `LoadBalancer` | A cloud load balancer with an external IP | Production external traffic on a cloud |
| `ExternalName` | DNS CNAME to an outside hostname, no proxying | Aliasing an external dependency |

They stack: `LoadBalancer` is a `NodePort` plus a cloud LB, and `NodePort` is a `ClusterIP` plus a node-wide port. In practice most cloud setups expose services through a **Gateway** object instead (later chapter).

## K8S-021 · `kubectl get endpoints`

- **First seen:** [[Ch4 Services]] → Services (assignment)
- **Purpose:** See which pod IPs a Service is currently routing to. The fastest way to prove a selector actually matches something.

```sh
kubectl get endpoints web-service
```

Output is one line: `NAME  ENDPOINTS  AGE`, where ENDPOINTS lists `10.1.2.3:8080,10.1.2.4:8080,...`. Three web pods means three entries on port `8080`.

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Target namespace |
| `-A` | All namespaces |
| `-o yaml` | Full object, including `subsets[].addresses` |

**Notes**

- `ep` is the short name.
- `ENDPOINTS  <none>` is the single most useful failure signal: the Service exists but matches zero **ready** pods. Check the selector, the pod labels, and whether the pods are Ready.
- Endpoints come from pods, so a pod that is running but failing its readiness probe is deliberately excluded.
- Newer clusters also expose `kubectl get endpointslices`, the scalable replacement, but `endpoints` is still fine for reading.
- If the `targetPort` is wrong you still see endpoints, just on a port nothing listens on. Traffic fails with connection refused.

## K8S-022 · `kubectl get services`

- **First seen:** [[Ch4 Services]] → Service Types (reading `clusterIP` and `type`)
- **Reused:** Ch4 · API Service (verifying `80:30080/TCP`) · [[Ch5 Namespaces]] → Creating a Namespace (`-n crawler get svc`) · [[Ch8 Health Checks]] → Readiness Probes · Broken Cluster · [[Ch10 Gateway]] → Tips (last hop in the routing ladder)
- **Purpose:** List Services, their type, ClusterIP and exposed ports.

```sh
kubectl get svc
```

Typical columns: `NAME  TYPE  CLUSTER-IP  EXTERNAL-IP  PORT(S)  AGE`.

**Reading PORT(S)**

| Value | Meaning |
|-------|---------|
| `8080/TCP` | ClusterIP only: internal port 8080 |
| `80:30080/TCP` | NodePort: port 80 inside, 30080 on every node |
| `80:31234/TCP` with an EXTERNAL-IP | LoadBalancer: node port plus a cloud IP |

**Flags**

| Flag | Effect |
|------|--------|
| `-o yaml` | Full spec, including the auto-assigned `clusterIP` and `type` |
| `-o wide` | Adds selector |
| `-n <namespace>` | Target namespace |
| `-A` | All namespaces |
| `-l app=<name>` | Filter by label |

**Notes**

- `svc` is an alias for `service`; both work everywhere.
- You never set `clusterIP` by hand. Kubernetes assigns it because `ClusterIP` is the default type, which is why a Service you wrote without a `type` still shows one.
- NodePorts live in `30000-32767` by default; asking for something outside that range is rejected unless the cluster was configured otherwise.
- `EXTERNAL-IP  <pending>` on a LoadBalancer usually means the cloud controller has no LB implementation, common on local clusters.
- `kubectl get svc <name> -o yaml` is how you inspect the generated DNS-relevant details, including `spec.clusterIP` and `spec.ports[].nodePort`.
- From inside the cluster, `http://web-service` works in the same namespace, `web-service.default` across namespaces, and the full `<svc>.<ns>.svc.cluster.local` always works.

## K8S-026 · `kubectl delete service`

- **First seen:** [[Ch5 Namespaces]] → Creating a Namespace (cleaning up the old `default` namespace copy)
- **Purpose:** Remove a Service, which stops routing traffic to its pods without touching the pods themselves.

```sh
kubectl delete service web-service
```

**Notes**

- The pods keep running. Only the stable address and its endpoints disappear, so anything calling the Service now fails to connect.
- `svc` works here too: `kubectl delete svc web-service`.
- Deleting and recreating a ClusterIP Service usually yields a **new ClusterIP**, so anything using the IP rather than the DNS name breaks. Prefer the name.
- In a move between namespaces, both copies can exist at once, which is why you delete the old one explicitly.

## Gateway API

The modern way to expose services. Four objects, each owned by a different team in practice:

| Object | Owner | Job |
|--------|-------|-----|
| `GatewayClass` | Infra | Names which controller (e.g. Envoy Gateway) implements gateways |
| `Gateway` | Infra | Declares the listener: protocol, port, entry point |
| `HTTPRoute` | App team | Hostnames and paths → backend Services |
| `Service` | App team | Stable address in front of pods |

Traffic path: client → Gateway-managed load balancer → Gateway → HTTPRoute rules → Service → Pod.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata: { name: app-gateway }
spec:
  gatewayClassName: app-gatewayclass
  listeners:
    - { name: http, protocol: HTTP, port: 80 }
```

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata: { name: web-httproute }
spec:
  parentRefs: [{ name: app-gateway }]
  hostnames: ["synchat.internal"]
  rules:
    - matches: [{ path: { type: PathPrefix, value: / } }]
      backendRefs: [{ name: web-service, port: 80 }]
```

`apiVersion: gateway.networking.k8s.io/v1` is not core Kubernetes. The Envoy Gateway install adds those CRDs, which is why `kubectl apply` must run before any Gateway manifest works.

**Ingress vs Gateway**

| | Ingress | Gateway API |
|---|---|---|
| Shape | One object for entry point + routes | GatewayClass + Gateway + HTTPRoutes |
| Extra behaviour | Controller-specific annotations | Standard fields |
| Status | Frozen | Actively developed |

Use Gateway for new work. You will still meet Ingress in existing systems.

## Intra-cluster DNS

Every Service gets a DNS name:

```text
<service-name>.<namespace>.svc.cluster.local
```

- Same namespace: `http://web-service`
- Cross namespace: `http://crawler.crawler` or the full `http://crawler.crawler.svc.cluster.local`
- `.svc.cluster.local` is usually optional; the short forms resolve first

Internal traffic is faster, needs no public DNS, and stays off the public internet, so keep it internal unless something outside the cluster must reach it.

## K8S-038 · `kubectl get gateway`

- **First seen:** [[Ch10 Gateway]] → Tunnel (reading the load balancer IP)
- **Reused:** Ch10 · DNS · Tips
- **Purpose:** Show Gateway objects and, critically, the address the load balancer handed to them.

```sh
kubectl get gateway app-gateway
```

Typical columns: `NAME  CLASS  ADDRESS  PROGRAMMED  AGE`.

**Pulling just the IP**

```sh
kubectl get gateway app-gateway -o custom-columns="IP:.status.addresses[0].value" --no-headers
```

`-o custom-columns` takes `Header:json.path` pairs; `--no-headers` drops the header so it can be used in a script or `/etc/hosts` edit.

**Flags**

| Flag | Effect |
|------|--------|
| `-o yaml` | Full status, including listeners and conditions |
| `-o custom-columns=...` | Pick specific fields |
| `-A` | All namespaces |
| `-n <namespace>` | Target namespace |

**Notes**

- An empty `ADDRESS` usually means the load balancer is not ready yet. On Minikube that means `minikube tunnel` is not running.
- `ADDRESS <none>` right after starting the tunnel is normal for a few seconds.
- The tunnel IP can change after a reboot, so re-check it before touching `/etc/hosts`.
- `kubectl get gateways` lists them; `get gatewayclass` shows the controller wiring.

## K8S-039 · `kubectl describe httproute`

- **First seen:** [[Ch10 Gateway]] → Tips (checking the route was `Accepted`)
- **Purpose:** See whether a route is actually attached to its Gateway and resolving its backends.

```sh
kubectl describe httproute web-httproute
```

**What to read**

| Section | Meaning |
|---------|---------|
| `Parents` | The Gateway this route attached to, with `Accepted` / `ResolvedRefs` conditions |
| `Rules` | Hostnames, matches, backend Services |
| `Events` | Controller messages, including rejection reasons |

**Notes**

- `Accepted: False` means the route is not bound to the Gateway. Check `parentRefs.name` and that both live in the same namespace.
- `ResolvedRefs: False` means a backend Service name or port is wrong.
- `get httproute` gives a one-line summary; `describe` is where the conditions live.
- A correct route with a wrong Service still fails at the next hop, so follow up with `kubectl get endpoints <service>`.

## Testing a route from your machine

With `minikube tunnel` open, the Gateway has a routable IP. Because routing is by `Host` header, `curl` must send one:

```sh
curl -H "Host: synchat.internal" http://<gateway-ip>
curl -H "Host: synchatapi.internal" http://<gateway-ip>/healthz
```

`ping <hostname>` only proves `/etc/hosts` resolves; it says nothing about routing. If ICMP is blocked, `curl` is the better test.

**Ladder when the site does not load** (chapter order, each step narrows the next):

1. `ping synchat.internal` resolves to the Gateway IP → else fix `/etc/hosts`
2. `kubectl get gateway app-gateway` has an address and the tunnel is running
3. `kubectl describe httproute web-httproute` shows `Accepted`
4. `kubectl get endpoints web-service` lists pod IPs
5. `kubectl logs` on the app pods shows requests arriving
