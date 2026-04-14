# LLM-D Conformance Manifests — RHAIIS 3.4 EA2

LLMInferenceService manifests for RHAIIS 3.4 EA2.

Based on downstream samples: [red-hat-data-services/kserve (rhoai-3.4-ea.2)](https://github.com/red-hat-data-services/kserve/tree/rhoai-3.4-ea.2/docs/samples/llmisvc)

## What's New in EA2

- **Simplified scheduler config** — `scheduler.config.inline` replaces verbose `scheduler.template.containers[].args` with `--config-text`
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

|------------|-----------------|-------|
| Manifest | Description |
|----------|-------------|
| `single-gpu.yaml` | 1 GPU with scheduler |
| `single-gpu-smoke.yaml` | 1 GPU, minimal smoke test |
| `single-gpu-no-scheduler.yaml` | 1 GPU, K8s native routing (no EPP) |
| `cache-aware.yaml` | Prefix KV cache-aware routing, 2 replicas |
| `pd.yaml` | P/D disaggregation |
| `pd-cache-aware.yaml` | P/D + cache-aware hybrid (EA2 only) |
| `moe.yaml` | MoE with DP/EP, 8 GPUs, RDMA/RoCE |


