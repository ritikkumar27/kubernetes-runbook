# Commands Index

Every command that appears in the chapter notes, extracted by hand and organised for real use.

| Commands | Category files | Chapters scanned | Last scan |
|----------|----------------|------------------|-----------|
| 42 | 15 | 12 | [[Ch12 Storage]] |

## Categories

| # | File | Label | Commands |
|---|------|-------|----------|
| 01 | [[01 - Cluster and Client Info]] | Cluster | 3 |
| 02 | [[02 - Viewing and Inspecting Resources]] | Inspect | 6 |
| 03 | [[03 - Workloads and Deployments]] | Workloads | 3 |
| 04 | [[04 - Pods]] | Pods | 1 |
| 05 | [[05 - Scaling]] | Scaling | 3 |
| 06 | [[06 - Logs and Debugging]] | Debug | 1 |
| 07 | [[07 - Services and Networking]] | Networking | 5 |
| 08 | [[08 - Configuration]] | Config | 4 |
| 09 | [[09 - Storage]] | Storage | 0 |
| 10 | [[10 - Declarative Manifests]] | Manifests | 1 |
| 11 | [[11 - Rollouts and Updates]] | Rollouts | 4 |
| 12 | [[12 - Namespaces and Contexts]] | Namespaces | 4 |
| 13 | [[13 - Nodes and Cluster Admin]] | Nodes | 2 |
| 14 | [[14 - Local Cluster and Tooling]] | Local | 4 |
| 15 | [[15 - Jobs and CronJobs]] | Jobs | 1 |

**09 Storage** is a concept reference (volumes, PV/PVC) with no commands yet: Chapter 12 changes storage entirely through YAML, and every command it uses is already indexed elsewhere.

## How this folder works

- **Serial numbers are permanent.** `K8S-001`, `K8S-002`, ... are assigned in the order the command *first appears* in the notes. A command keeps its number forever.
- **Categories follow real kubectl usage** (inspect → change → debug → scale), not the docs sidebar. Each category is its own numbered file.
- **One entry per command**, not per invocation. Repeats across chapters are added to the entry's **Reused** line.

## Debugging ladder

Work from a high-level symptom down to the root cause. Each step narrows the next one.

| Step | Command | What it tells you | Go here when |
|------|---------|-------------------|--------------|
| 1 | `kubectl get pods` | Which pods are not `Running` and `1/1` | Always start here |
| 2 | `kubectl describe pod` | Events, last state, probe config | A specific pod is unhealthy |
| 3 | `kubectl logs` | Why the app died | `RESTARTS` is climbing |
| 4 | `kubectl get endpoints` | Which pods actually receive traffic | Pods look fine but traffic fails |

Two traps this ladder catches:

- A pod `Running` but `0/1` with `RESTARTS 0` is a **readiness** problem, not a crash. Do not read logs; read the probe config.
- A Service whose selector matches nothing is perfectly valid, so Kubernetes never warns you. `endpoints` showing `<none>` is the only signal.

## Master index

Serial order, exactly as assigned. Category labels match the table above; open the file there.

| Serial | Command | Category | First seen |
|--------|---------|----------|------------|
| K8S-001 | `kubectl version` | Cluster | [[Ch1 Pods]] |
| K8S-002 | `kubectl create deployment` | Workloads | [[Ch1 Pods]] |
| K8S-003 | `kubectl get deployments` | Inspect | [[Ch1 Pods]] |
| K8S-004 | `kubectl get pods` | Inspect | [[Ch1 Pods]] |
| K8S-005 | `kubectl scale deployment` | Scaling | [[Ch1 Pods]] |
| K8S-006 | `kubectl logs` | Debug | [[Ch1 Pods]] |
| K8S-007 | `kubectl delete pod` | Pods | [[Ch1 Pods]] |
| K8S-008 | `kubectl edit deployment` | Workloads | [[Ch1 Pods]] |
| K8S-009 | `kubectl describe pod` | Inspect | [[Ch1 Pods]] |
| K8S-010 | `kubectl describe deployment` | Inspect | [[Ch1 Pods]] |
| K8S-011 | `kubectl get replicasets` | Inspect | [[Ch1 Pods]] |
| K8S-012 | `kubectl apply -f` | Manifests | [[Ch1 Pods]] |
| K8S-013 | `kubectl explain` | Cluster | [[Ch2 Pod Management]] |
| K8S-014 | `kubectl rollout status deployment` | Rollouts | [[Ch2 Pod Management]] |
| K8S-015 | `kubectl rollout history deployment` | Rollouts | [[Ch2 Pod Management]] |
| K8S-016 | `kubectl rollout restart deployment` | Rollouts | [[Ch2 Pod Management]] |
| K8S-017 | `kubectl rollout undo deployment` | Rollouts | [[Ch2 Pod Management]] |
| K8S-018 | `kubectl get configmaps` | Config | [[Ch3 ConfigMaps]] |
| K8S-019 | `kubectl create secret generic` | Config | [[Ch3 ConfigMaps]] |
| K8S-020 | `kubectl get secret` | Config | [[Ch3 ConfigMaps]] |
| K8S-021 | `kubectl get endpoints` | Networking | [[Ch4 Services]] |
| K8S-022 | `kubectl get services` | Networking | [[Ch4 Services]] |
| K8S-023 | `kubectl create namespace` | Namespaces | [[Ch5 Namespaces]] |
| K8S-024 | `kubectl get namespaces` | Namespaces | [[Ch5 Namespaces]] |
| K8S-025 | `kubectl delete deployment` | Workloads | [[Ch5 Namespaces]] |
| K8S-026 | `kubectl delete service` | Networking | [[Ch5 Namespaces]] |
| K8S-027 | `kubectl delete configmap` | Config | [[Ch5 Namespaces]] |
| K8S-028 | `kubectl config get-contexts` | Namespaces | [[Ch5 Namespaces]] |
| K8S-029 | `kubectl config set-context --current` | Namespaces | [[Ch5 Namespaces]] |
| K8S-030 | `kubectl top pod` | Inspect | [[Ch6 Resources]] |
| K8S-031 | `kubectl get hpa` | Scaling | [[Ch7 Scaling]] |
| K8S-032 | `kubectl describe hpa` | Scaling | [[Ch7 Scaling]] |
| K8S-033 | `kubectl get nodes` | Nodes | [[Ch7 Scaling]] |
| K8S-034 | `minikube addons enable` | Local | [[Ch9 Local Cluster]] |
| K8S-035 | `kubectl top node` | Nodes | [[Ch9 Local Cluster]] |
| K8S-036 | `kubectl proxy` | Cluster | [[Ch9 Local Cluster]] |
| K8S-037 | `minikube tunnel` | Local | [[Ch10 Gateway]] |
| K8S-038 | `kubectl get gateway` | Networking | [[Ch10 Gateway]] |
| K8S-039 | `kubectl describe httproute` | Networking | [[Ch10 Gateway]] |
| K8S-040 | `kubectl get jobs` | Jobs | [[Ch11 Jobs]] |
| K8S-041 | `minikube stop` | Local | [[Ch12 Storage]] |
| K8S-042 | `minikube delete` | Local | [[Ch12 Storage]] |

## Scan log

One row per scan. Chapters that only reused existing commands record `none`.

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
| Ch8 Health Checks | 2026-11-22 | none | Probes documented under 04 Pods; debugging ladder added here |
| Ch9 Local Cluster | 2026-11-23 | K8S-034 … K8S-036 | minikube addons, `top node`, `proxy`, install notes |
| Ch10 Gateway | 2026-11-23 | K8S-037 … K8S-039 | Gateway API, HTTPRoute conditions, tunnel |
| Ch11 Jobs | 2026-11-23 | K8S-040 | Jobs, CronJobs, cron syntax |
| Ch12 Storage | 2026-11-23 | K8S-041 … K8S-042 | Volumes, PV/PVC concepts in 09; `minikube stop` / `delete` |
