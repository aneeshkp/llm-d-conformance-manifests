# LLM-D Conformance Manifests — RHAIIS 3.4

LLMInferenceService manifests for RHAIIS 3.4 conformance testing.

Based on downstream samples: [red-hat-data-services/kserve (rhoai-3.4)](https://github.com/red-hat-data-services/kserve/tree/rhoai-3.4/docs/samples/llmisvc)

## What's New in 3.4 (from EA2)

- **Pull secret renamed** — `rhai-pull-secret` (was `rhaii-pull-secret` in EA2). Matches the name created by the `rhai-on-xks-chart`.
- **New manifests** — `pd-cache-aware.yaml` (P/D + cache-aware hybrid), `multi-pool.yaml` (multi-pool VirtualService merge)
- **Simplified scheduler config** — `scheduler.config.inline` replaces verbose `scheduler.template.containers[].args` with `--config-text`
- **Cache-aware routing** — uses `precise-prefix-cache-scorer` plugin configured via `scheduler.config.inline`

## Branch Strategy

| Branch | Version | Pull Secret | Notes |
|--------|---------|-------------|-------|
| `main` / `3.4-stable` | RHAIIS 3.4 (latest stable) | `rhai-pull-secret` | Current |
| `3.4-ea2` | RHAIIS 3.4 EA2 | `rhaii-pull-secret` | Frozen |
| `3.4-ea1` | RHAIIS 3.4 EA1 | `redhat-pull-secret` | Frozen, uses `--config-text` args format |

## Key Notes

### Pull secrets

All manifests use `rhai-pull-secret`. This matches the `rhai-on-xks-chart` which creates `rhai-pull-secret` in all namespaces.

### Scheduler template with container stubs

When providing `scheduler.template.imagePullSecrets` in the manifest, you **must** also include container name stubs. Otherwise KServe replaces the entire template from `LLMInferenceServiceConfig` and the scheduler deployment fails with `spec.template.spec.containers: Required value`.

```yaml
scheduler:
  template:
    imagePullSecrets:
    - name: rhai-pull-secret
    containers:
    - name: main        # scheduler (EPP)
    - name: tokenizer   # kv-cache / tokenizer sidecar
```

### Plugin name change (since EA1)

The prefix cache scorer plugin was renamed:
- **EA1:** `prefix-cache-scorer`
- **EA2+:** `precise-prefix-cache-scorer`

## Manifests

| Manifest | Description | Scheduler |
|----------|-------------|-----------|
| `single-gpu.yaml` | 1 GPU with scheduler | template + stubs |
| `single-gpu-smoke.yaml` | 1 GPU, minimal smoke test | template + stubs |
| `single-gpu-no-scheduler.yaml` | 1 GPU, K8s native routing (no EPP) | none |
| `cache-aware.yaml` | Prefix KV cache-aware routing, 2 replicas | template + stubs + config.inline |
| `pd.yaml` | P/D disaggregation | template + stubs |
| `pd-cache-aware.yaml` | P/D + cache-aware hybrid | template + stubs + config.inline |
| `multi-pool.yaml` | Multi-pool VirtualService merge test | template + stubs |
| `moe.yaml` | MoE with DP/EP, 8 GPUs, RDMA/RoCE | template + stubs |
