# LLM-D Conformance Manifests — RHAIIS 3.4 EA2

LLMInferenceService manifests for RHAIIS 3.4 EA2.

## What's New in EA2

- **Simplified scheduler config** — `scheduler.config.inline` replaces verbose `scheduler.template.containers[].args` with `--config-text`
- **P/D routing** — `scheduler: {}` is sufficient, CRD handles defaults
- **Cache-aware routing** — uses `precise-prefix-cache-scorer` plugin (renamed from `prefix-cache-scorer` in EA1) configured via `scheduler.config.inline`
- **New CRD field** — `scheduler.config.inline` added to the LLMInferenceService CRD

## Key Differences from EA1

### Pull secrets

All manifests use `rhaii-pull-secret` (not `redhat-pull-secret`). This matches the operator-based deployment (`rhai-on-xks-chart`) which creates `rhaii-pull-secret` in all namespaces.

### Scheduler template with container stubs

When providing `scheduler.template.imagePullSecrets` in the manifest, you **must** also include container name stubs. Otherwise KServe replaces the entire template from `LLMInferenceServiceConfig` and the scheduler deployment fails with `spec.template.spec.containers: Required value`.

```yaml
scheduler:
  template:
    imagePullSecrets:
    - name: rhaii-pull-secret
    containers:
    - name: main        # scheduler (EPP)
    - name: tokenizer   # kv-cache / tokenizer sidecar
```

KServe merges these stubs with the full container specs from the `LLMInferenceServiceConfig`. Without the stubs, the containers field is empty and the deployment is rejected.

### Plugin name change

The prefix cache scorer plugin was renamed:
- **EA1:** `prefix-cache-scorer`
- **EA2:** `precise-prefix-cache-scorer`

The `schedulingProfiles[].plugins[].pluginRef` must match the registered plugin name exactly.

## Manifests

| Manifest | Description | Pull secret | Scheduler |
|----------|-------------|-------------|-----------|
| `single-gpu.yaml` | 1 GPU with scheduler | `rhaii-pull-secret` | template + stubs |
| `single-gpu-smoke.yaml` | 1 GPU, minimal smoke test | `rhaii-pull-secret` | template + stubs |
| `single-gpu-no-scheduler.yaml` | 1 GPU, K8s native routing (no EPP) | `rhaii-pull-secret` | none |
| `cache-aware.yaml` | Prefix KV cache-aware routing, 2 replicas | `rhaii-pull-secret` | template + stubs + config.inline |
| `pd.yaml` | P/D disaggregation | `rhaii-pull-secret` | template + stubs |
| `moe.yaml` | MoE with DP/EP, 8 GPUs, RDMA/RoCE | `rhaii-pull-secret` | template + stubs |

## Deployment types

These manifests work with two deployment models:

| Deployment | Pull secret name | Chart |
|------------|-----------------|-------|
| **Operator-based** (`rhai-on-xks-chart`) | `rhaii-pull-secret` (default) | `odh-gitops` |
| **Helmfile-based** (`rhaii-on-xks`) | Override with `PULL_SECRET=redhat-pull-secret` | `rhaii-on-xks` |

## How to check your cluster version

```bash
kubectl get crd llminferenceservices.serving.kserve.io -o json | \
  jq '.spec.versions[].schema.openAPIV3Schema.properties.spec.properties.router.properties.scheduler.properties.config'
# Returns null → EA1 (use 3.4-ea1 branch), returns object → EA2 (this branch)
```
