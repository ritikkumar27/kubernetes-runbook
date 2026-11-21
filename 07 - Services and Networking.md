# 07 · Services and Networking

Pods come and go with new IPs. A Service is the stable address in front of them.

| Serial | Command |
|--------|---------|
| K8S-021 | `kubectl get endpoints` |
| K8S-022 | `kubectl get services` |

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
- **Reused:** Ch4 · API Service (verifying `80:30080/TCP`)
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
