# Commands Index

Every `kubectl` command that appears in the chapter notes, extracted and organised.

## How this folder works

- **Serial numbers are permanent.** `K8S-001`, `K8S-002`, ... are assigned in the order the command *first appears* in the notes. A command keeps its number forever.
- **Categories follow real kubectl usage** (inspect → change → debug → scale), not the docs sidebar. Each category is its own numbered file.
- **One entry per command**, not per invocation. Repeats across chapters are added to the entry's **Reused** line.

## Debugging ladder

Work from a high-level symptom down to the root cause. This order is fast because each step narrows the next one.

| Step | Command | What it tells you | Go here when |
|------|---------|-------------------|--------------|
| 1 | `kubectl get pods` | Which pods are not `Running` and `1/1` | Always start here |
| 2 | `kubectl describe pod` | Events, last state, probe config | A specific pod is unhealthy |
| 3 | `kubectl logs` | Why the app died | `RESTARTS` is climbing |
| 4 | `kubectl get endpoints` | Which pods actually receive traffic | Pods look fine but traffic fails |

Two traps this ladder catches:

- A pod `Running` but `0/1` with `RESTARTS 0` is a **readiness** problem, not a crash. Do not read logs; read the probe config.
- A Service whose selector matches nothing is perfectly valid, so Kubernetes never warns you. `endpoints` showing `<none>` is the only signal.

## At a glance

| Category | File | Commands |
|----------|------|----------|
| 01 | [[01 - Cluster and Client Info]] | 3 |
| 02 | [[02 - Viewing and Inspecting Resources]] | 6 |
| 03 | [[03 - Workloads and Deployments]] | 3 |
| 04 | [[04 - Pods]] | 1 |
| 05 | [[05 - Scaling]] | 3 |
| 06 | [[06 - Logs and Debugging]] | 1 |
| 07 | [[07 - Services and Networking]] | 5 |
| 08 | [[08 - Configuration]] | 4 |
| 09 | [[09 - Storage]] | 0 |
| 10 | [[10 - Declarative Manifests]] | 1 |
| 11 | [[11 - Rollouts and Updates]] | 4 |
| 12 | [[12 - Namespaces and Contexts]] | 4 |
| 13 | [[13 - Nodes and Cluster Admin]] | 2 |
| 14 | [[14 - Local Cluster and Tooling]] | 4 |
| 15 | [[15 - Jobs and CronJobs]] | 1 |

**Total:** 42 commands · Last scanned: Ch12 Storage

09 Storage holds a concept reference (volumes, PV/PVC) but no commands yet: Chapter 12 changes storage entirely through YAML, and every command it uses is already indexed elsewhere.

## Master index

| Serial | Command | Category | First seen |
|--------|---------|----------|------------|
| K8S-001 | `kubectl version` | Cluster and Client Info | [[Ch1 Pods]] |
| K8S-002 | `kubectl create deployment` | Workloads and Deployments | [[Ch1 Pods]] |
| K8S-003 | `kubectl get deployments` | Viewing and Inspecting Resources | [[Ch1 Pods]] |
| K8S-004 | `kubectl get pods` | Viewing and Inspecting Resources | [[Ch1 Pods]] |
| K8S-005 | `kubectl scale deployment` | Scaling | [[Ch1 Pods]] |
| K8S-006 | `kubectl logs` | Logs and Debugging | [[Ch1 Pods]] |
| K8S-007 | `kubectl delete pod` | Pods | [[Ch1 Pods]] |
| K8S-008 | `kubectl edit deployment` | Workloads and Deployments | [[Ch1 Pods]] |
| K8S-009 | `kubectl describe pod` | Viewing and Inspecting Resources | [[Ch1 Pods]] |
| K8S-010 | `kubectl describe deployment` | Viewing and Inspecting Resources | [[Ch1 Pods]] |
| K8S-011 | `kubectl get replicasets` | Viewing and Inspecting Resources | [[Ch1 Pods]] |
| K8S-012 | `kubectl apply -f` | Declarative Manifests | [[Ch1 Pods]] |
| K8S-013 | `kubectl explain` | Cluster and Client Info | [[Ch2 Pod Management]] |
| K8S-014 | `kubectl rollout status deployment` | Rollouts and Updates | [[Ch2 Pod Management]] |
| K8S-015 | `kubectl rollout history deployment` | Rollouts and Updates | [[Ch2 Pod Management]] |
| K8S-016 | `kubectl rollout restart deployment` | Rollouts and Updates | [[Ch2 Pod Management]] |
| K8S-017 | `kubectl rollout undo deployment` | Rollouts and Updates | [[Ch2 Pod Management]] |
| K8S-018 | `kubectl get configmaps` | Configuration | [[Ch3 ConfigMaps]] |
| K8S-019 | `kubectl create secret generic` | Configuration | [[Ch3 ConfigMaps]] |
| K8S-020 | `kubectl get secret` | Configuration | [[Ch3 ConfigMaps]] |
| K8S-021 | `kubectl get endpoints` | Services and Networking | [[Ch4 Services]] |
| K8S-022 | `kubectl get services` | Services and Networking | [[Ch4 Services]] |
| K8S-023 | `kubectl create namespace` | Namespaces and Contexts | [[Ch5 Namespaces]] |
| K8S-024 | `kubectl get namespaces` | Namespaces and Contexts | [[Ch5 Namespaces]] |
| K8S-025 | `kubectl delete deployment` | Workloads and Deployments | [[Ch5 Namespaces]] |
| K8S-026 | `kubectl delete service` | Services and Networking | [[Ch5 Namespaces]] |
| K8S-027 | `kubectl delete configmap` | Configuration | [[Ch5 Namespaces]] |
| K8S-028 | `kubectl config get-contexts` | Namespaces and Contexts | [[Ch5 Namespaces]] |
| K8S-029 | `kubectl config set-context --current` | Namespaces and Contexts | [[Ch5 Namespaces]] |
| K8S-030 | `kubectl top pod` | Viewing and Inspecting Resources | [[Ch6 Resources]] |
| K8S-031 | `kubectl get hpa` | Scaling | [[Ch7 Scaling]] |
| K8S-032 | `kubectl describe hpa` | Scaling | [[Ch7 Scaling]] |
| K8S-033 | `kubectl get nodes` | Nodes and Cluster Admin | [[Ch7 Scaling]] |
| K8S-034 | `minikube addons enable` | Local Cluster and Tooling | [[Ch9 Local Cluster]] |
| K8S-035 | `kubectl top node` | Nodes and Cluster Admin | [[Ch9 Local Cluster]] |
| K8S-036 | `kubectl proxy` | Cluster and Client Info | [[Ch9 Local Cluster]] |
| K8S-037 | `minikube tunnel` | Local Cluster and Tooling | [[Ch10 Gateway]] |
| K8S-038 | `kubectl get gateway` | Services and Networking | [[Ch10 Gateway]] |
| K8S-039 | `kubectl describe httproute` | Services and Networking | [[Ch10 Gateway]] |
| K8S-040 | `kubectl get jobs` | Jobs and CronJobs | [[Ch11 Jobs]] |
| K8S-041 | `minikube stop` | Local Cluster and Tooling | [[Ch12 Storage]] |
| K8S-042 | `minikube delete` | Local Cluster and Tooling | [[Ch12 Storage]] |

## Concept-only files

`09 - Storage` exists because Chapter 12 explains volumes, PVs and PVCs without introducing a single new `kubectl` resource command. If a later chapter adds one, it becomes a normal entry there.

## Scan log

| Chapter | Scanned | New commands | Notes |
|---------|---------|--------------|-------|
| Ch1 Pods | 2026-11-18 | K8S-001 … K8S-008 | Initial build |
| Ch1 Pods (part 2) | 2026-11-18 | K8S-009 … K8S-012 | Describe, ReplicaSets, YAML config, apply |
| Ch2 Pod Management | 2026-11-18 | K8S-013 … K8S-017 | Explain, rollout status/history/restart/undo |
| Ch2 (part 2) | 2026-11-19 | none | API Deployment, Thrashing Pods: reuse only |
| Ch3 ConfigMaps | 2026-11-19 | K8S-018 … K8S-020 | ConfigMaps, Secrets, decoding values |
| Ch3 (part 2) | 2026-11-20 | none | Crawler, YAML types: reuse only. Ch1 note renamed to `Ch1 Pods.md`, links updated |
| Ch4 Services | 2026-11-21 | K8S-021 … K8S-022 | Endpoints, service types, NodePort |
| Ch5 Namespaces | 2026-11-21 | K8S-023 … K8S-029 | Namespaces, deletes, contexts |
| Ch6 Resources | 2026-11-21 | K8S-030 | `top`, limits vs requests, OOMKilled vs Pending |
| Ch7 Scaling | 2026-11-21 | K8S-031 … K8S-033 | HPA get/describe, nodes |
| Ch8 Health Checks | 2026-11-22 | none | Probes documented under 04 Pods; debugging ladder added to README |
| Ch9 Local Cluster | 2026-11-23 | K8S-034 … K8S-036 | minikube addons, `top node`, `proxy`, install notes |
| Ch10 Gateway | 2026-11-23 | K8S-037 … K8S-039 | Gateway API, HTTPRoute conditions, tunnel |
| Ch11 Jobs | 2026-11-23 | K8S-040 | Jobs, CronJobs, cron syntax |
| Ch12 Storage | 2026-11-23 | K8S-041 … K8S-042 | Volumes, PV/PVC concepts in 09; `minikube stop` / `delete` |
