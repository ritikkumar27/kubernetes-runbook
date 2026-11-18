# Commands Index

Every `kubectl` command that appears in the chapter notes, extracted and organised.

## How this folder works

- **Serial numbers are permanent.** `K8S-001`, `K8S-002`, ... are assigned in the order the command *first appears* in the notes. A command keeps its number forever.
- **Categories follow real kubectl usage** (inspect → change → debug → scale), not the docs sidebar. Each category is its own numbered file.
- **One entry per command**, not per invocation. Repeats across chapters are added to the entry's **Reused** line.

## At a glance

| Category | File | Commands |
|----------|------|----------|
| 01 | [[01 - Cluster and Client Info]] | 1 |
| 02 | [[02 - Viewing and Inspecting Resources]] | 5 |
| 03 | [[03 - Workloads and Deployments]] | 2 |
| 04 | [[04 - Pods]] | 1 |
| 05 | [[05 - Scaling]] | 1 |
| 06 | [[06 - Logs and Debugging]] | 1 |
| 10 | [[10 - Declarative Manifests]] | 1 |

**Total:** 12 commands · Last scanned: Ch1 Pods (complete)

## Master index

| Serial | Command | Category | First seen |
|--------|---------|----------|------------|
| K8S-001 | `kubectl version` | Cluster and Client Info | [[Ch1 Pods - 1]] |
| K8S-002 | `kubectl create deployment` | Workloads and Deployments | [[Ch1 Pods - 1]] |
| K8S-003 | `kubectl get deployments` | Viewing and Inspecting Resources | [[Ch1 Pods - 1]] |
| K8S-004 | `kubectl get pods` | Viewing and Inspecting Resources | [[Ch1 Pods - 1]] |
| K8S-005 | `kubectl scale deployment` | Scaling | [[Ch1 Pods - 1]] |
| K8S-006 | `kubectl logs` | Logs and Debugging | [[Ch1 Pods - 1]] |
| K8S-007 | `kubectl delete pod` | Pods | [[Ch1 Pods - 1]] |
| K8S-008 | `kubectl edit deployment` | Workloads and Deployments | [[Ch1 Pods - 1]] |
| K8S-009 | `kubectl describe pod` | Viewing and Inspecting Resources | [[Ch1 Pods - 1]] |
| K8S-010 | `kubectl describe deployment` | Viewing and Inspecting Resources | [[Ch1 Pods - 1]] |
| K8S-011 | `kubectl get replicasets` | Viewing and Inspecting Resources | [[Ch1 Pods - 1]] |
| K8S-012 | `kubectl apply -f` | Declarative Manifests | [[Ch1 Pods - 1]] |

## Reserved categories

Not created yet. A file appears only when a scanned chapter contains a command for it.

| # | Scope |
|---|-------|
| 07 | Services and Networking |
| 08 | Configuration (ConfigMaps / Secrets) |
| 09 | Storage (Volumes / PVCs) |
| 11 | Rollouts and Updates |
| 12 | Namespaces and Contexts |
| 13 | Nodes and Cluster Admin |

## Scan log

| Chapter | Scanned | New commands | Notes |
|---------|---------|--------------|-------|
| Ch1 Pods | 2026-11-18 | K8S-001 … K8S-008 | Initial build |
| Ch1 Pods (part 2) | 2026-11-18 | K8S-009 … K8S-012 | Describe, ReplicaSets, YAML config, apply |
