# 08 · Configuration

Config that lives outside the container image: ConfigMaps for innocent values, Secrets for sensitive ones.

| Serial | Command |
|--------|---------|
| K8S-018 | `kubectl get configmaps` |
| K8S-019 | `kubectl create secret generic` |
| K8S-020 | `kubectl get secret` |
| K8S-027 | `kubectl delete configmap` |

## Why ConfigMaps exist

An `ENV` in a Dockerfile bakes a value into the image, so changing it means rebuilding. A ConfigMap is a separate object holding key/value strings, shared across deployments and edited without touching the image.

Values must be **strings**, so quote numbers: `API_PORT: "8080"`.

## K8S-018 · `kubectl get configmaps`

- **First seen:** [[Ch3 ConfigMaps]] → Config Maps (assignment)
- **Reused:** [[Ch5 Namespaces]] → Creating a Namespace (`-n crawler get configmaps`)
- **Purpose:** List ConfigMaps in the namespace and confirm yours was created.

```sh
kubectl get configmaps
```

Typical columns: `NAME  DATA  AGE`. `DATA` is the number of keys in the map.

**Flags**

| Flag | Effect |
|------|--------|
| `-n <namespace>` | Target namespace |
| `-A` | All namespaces |
| `-o yaml` | Full object, including every key and value |
| `-l app=<name>` | Filter by label |

**Short form and reading values**

```sh
kubectl get cm synergychat-api-configmap -o yaml
kubectl describe configmap synergychat-api-configmap
```

`cm` is the short name. Plain `get` hides the values, `-o yaml` shows them raw, `describe` prints them in a readable block.

**Consuming a ConfigMap in a Deployment**

```yaml
env:
  - name: API_PORT
    valueFrom:
      configMapKeyRef:
        name: synergychat-api-configmap
        key: API_PORT
```

One key at a time with `configMapKeyRef`. For a whole map, use `envFrom` instead:

```yaml
envFrom:
  - configMapRef:
      name: synergychat-crawler-configmap
```

That injects **every key** in the map as an environment variable, which beats repeating the verbose block per variable.

**Edits do not reach running pods.** Environment variables are set when a container starts. `apply`ing a changed ConfigMap changes nothing inside a running pod, and does not trigger a rollout. Run `kubectl rollout restart deployment <name>` to replace the pods and pick up the new values.

A missing map or key stops the pod before it starts (`CreateContainerConfigError`).

## K8S-019 · `kubectl create secret generic`

- **First seen:** [[Ch3 ConfigMaps]] → Secrets (assignment)
- **Purpose:** Create a Secret from literals or files, the usual way since secret YAML should not be committed.

```sh
kubectl create secret generic synergychat-api-secret --from-literal=AI_API_KEY=sk-totally-real-key
```

**Flags**

| Flag | Effect |
|------|--------|
| `--from-literal=KEY=VALUE` | Inline value, repeatable |
| `--from-file=path` | Whole file as one value, key defaults to the file name |
| `--from-file=KEY=path` | Explicit key for a file |
| `--from-env-file=file.env` | One key per `KEY=VALUE` line |
| `--type=<type>` | Default `Opaque`. Also `docker-registry`, `tls` |
| `--dry-run=client -o yaml` | Generate the manifest without creating |
| `-n <namespace>` | Target namespace |

**Notes**

- Values are stored **base64 encoded, which is not encryption**. Anyone allowed to read the Secret can decode it.
- `kubectl describe secret` does not print values; `get -o yaml` does. RBAC can give Secrets tighter permissions than ConfigMaps, which is the real reason they are separate objects.
- Neither ConfigMaps nor Secrets are encrypted at rest by default. That is a cluster-admin setting.
- Keep Secrets out of Git. Create them with `kubectl create secret` or sync them from a secret manager (Vault, AWS Secrets Manager).
- In YAML, `data` requires base64 values; `stringData` accepts plain text and is encoded on the way in.

**Consuming a Secret**

```yaml
env:
  - name: AI_API_KEY
    valueFrom:
      secretKeyRef:
        name: synergychat-api-secret
        key: AI_API_KEY
```

`secretKeyRef` instead of `configMapKeyRef`, everything else identical.

## K8S-020 · `kubectl get secret`

- **First seen:** [[Ch3 ConfigMaps]] → Secrets (assignment)
- **Purpose:** Inspect a Secret's YAML, then decode a single value when you actually need it.

```sh
kubectl get secret synergychat-api-secret -o yaml
```

**Decoding one key**

```sh
kubectl get secret synergychat-api-secret -o jsonpath='{.data.AI_API_KEY}' | base64 -d
```

**Flags**

| Flag | Effect |
|------|--------|
| `-o yaml` | Full object, `data` values in base64 |
| `-o jsonpath='{.data.KEY}'` | One value, still base64, ready to pipe |
| `--template='{{.data.KEY | base64decode}}'` | Decode inside kubectl |
| `-n <namespace>` | Target namespace |

**Notes**

- `.data` is base64. `.stringData` only exists on input, so you never see it in a `get`.
- `kubectl describe secret` shows key names and byte sizes, never values. Good: it is safe to paste into a ticket.
- Base64 output has no newline, so `base64 -d` works directly. Add `-w0` on macOS, or use the `--template` form, which is portable.
- `kubectl get secrets` (plural) lists them, same columns as ConfigMaps.

## When config is missing

| Symptom | Cause |
|---------|-------|
| Pod stuck in `CreateContainerConfigError` | Referenced ConfigMap/Secret or key does not exist |
| No logs at all | The container never started, so there is nothing to print |
| `describe pod` events say `couldn't find key X in ConfigMap ...` | That is the exact missing key |

Fix by pointing the container at an existing key, or by adding the key to the ConfigMap. Unlike a crash, there is no app output to read, so events are the only evidence.

## YAML types

YAML infers a type for every value, and Kubernetes is strict about it.

| Written | Parsed as | Valid in `data` / env `value`? |
|---------|-----------|-------------------------------|
| `"8080"` | string | yes |
| `8080` | number | no |
| `"true"` / `"yes"` | string | yes |
| `true` / `yes` | boolean | no |
| `"1.10"` | string | yes |
| `1.10` | float | no |

Every ConfigMap `data` value must be a string, and so must an env var's `value`. Get it wrong and the whole file is rejected:

```text
json: cannot unmarshal number into Go struct field ConfigMap.data of type string
```

Fields like `spec.replicas` expect a real number, so leave those unquoted.

**Rule:** if a value looks like a number or boolean but must be a string, quote it. That includes `"8080"`, `"true"`, `"1.10"`, `"yes"`, and the keywords string `"love,hate,joy"`. The parser also treats `yes`, `no`, `on` and `off` as booleans, which is a well-known source of confusion.

## K8S-027 · `kubectl delete configmap`

- **First seen:** [[Ch5 Namespaces]] → Creating a Namespace (cleaning up the old `default` namespace copy)
- **Purpose:** Remove a ConfigMap.

```sh
kubectl delete configmap synergychat-crawler-configmap
```

**Notes**

- Pods already running keep their environment variables. The values were copied in at container start, so nothing breaks immediately.
- The damage shows up on the **next** restart: the pod cannot build its environment and lands in `CreateContainerConfigError`. Since a Deployment replaces pods on any rollout, that failure is often one `apply` away.
- `cm` is the short form. `delete secret` behaves the same way.
- If the ConfigMap is mounted as a volume instead of env vars, files can vanish from the running pod, which is a different and nastier failure mode.
