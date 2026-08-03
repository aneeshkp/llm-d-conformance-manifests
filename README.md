# LLM-D Conformance Manifests — RHAIIS 3.5

LLMInferenceService manifests for RHAIIS 3.5 conformance testing.

Based on downstream samples: [red-hat-data-services/kserve (rhoai-3.5)](https://github.com/red-hat-data-services/kserve/tree/rhoai-3.5/docs/samples/llmisvc)

## What's New in 3.5 (from 3.4)

- **v1alpha2 API** — manifests use `serving.kserve.io/v1alpha2` (was `v1alpha1` in 3.4).
- **Scheduler nested under `router`** — the scheduler now lives at `spec.router.scheduler` alongside `router.route` and `router.gateway` (was top-level `spec.scheduler`).
- **No `tokenizer` sidecar in the scheduler** — the 3.5 scheduler preset (`config-llm-scheduler.yaml`) defines only the `main` (EPP) container. The `tokenizer` container stub used in 3.4 has been removed from all manifests.
- **New flow-control manifests** — `flow-control.yaml` and `flow-control-tokens.yaml` exercise the `flowControl` feature gate and saturation detection.
- **Cache-aware routing split into two plugins** — `precise-prefix-cache-producer` (KV event indexer) + `prefix-cache-scorer` (references the producer via `prefixMatchInfoProducerName`), replacing the single `precise-prefix-cache-scorer` used in 3.4.

## Branch Strategy

| Branch | Version | Pull Secret | API | Notes |
|--------|---------|-------------|-----|-------|
| `main` | RHAIIS 3.5 (latest) | `rhai-pull-secret` | `v1alpha2` | Current |
| `3.4-stable` | RHAIIS 3.4 GA | `rhai-pull-secret` | `v1alpha1` | Frozen |
| `3.4-ea2` | RHAIIS 3.4 EA2 | `rhaii-pull-secret` | `v1alpha1` | Frozen |
| `3.4-ea1` | RHAIIS 3.4 EA1 | `redhat-pull-secret` | `v1alpha1` | Frozen, uses `--config-text` args format |

## Key Notes

### Pull secrets

All manifests use `rhai-pull-secret`. This matches the `rhai-on-xks-chart` which creates `rhai-pull-secret` in all namespaces.

### Scheduler template with container stubs

When providing `router.scheduler.template.imagePullSecrets` in the manifest, you **must** also include the container name stub. Otherwise KServe replaces the entire template from `LLMInferenceServiceConfig` and the scheduler deployment fails with `spec.template.spec.containers: Required value`.

The scheduler preset (`config-llm-scheduler.yaml`) defines a single container, `main` (the EPP). In 3.5 there is no `tokenizer` sidecar in the scheduler template, so only the `main` stub is needed:

```yaml
router:
  scheduler:
    template:
      imagePullSecrets:
      - name: rhai-pull-secret
      containers:
      - name: main        # scheduler (EPP)
  route: {}
  gateway: {}
```

### Cache-aware plugin configuration (since 3.5)

Cache-aware routing now uses two cooperating plugins configured via `router.scheduler.config.inline`:

- `precise-prefix-cache-producer` — consumes vLLM KV cache events (ZMQ) and builds the block index.
- `prefix-cache-scorer` — scores endpoints, pointing at the producer with `prefixMatchInfoProducerName: precise-prefix-cache-producer`.

In 3.4 this was a single `precise-prefix-cache-scorer` plugin.

## Manifests

| Manifest | Description | Scheduler |
|----------|-------------|-----------|
| `single-gpu.yaml` | 1 GPU with scheduler | template + stubs |
| `single-gpu-smoke.yaml` | 1 GPU, minimal smoke test | template + stubs |
| `single-gpu-no-scheduler.yaml` | 1 GPU, K8s native routing (no EPP) | none |
| `cache-aware.yaml` | Prefix KV cache-aware routing, 2 replicas | template + stubs + config.inline |
| `flow-control.yaml` | Flow control feature gate, default request TTL | template + stubs + config.inline |
| `flow-control-tokens.yaml` | Flow control with token-based concurrency + saturation detection | template + stubs + config.inline |
| `pd.yaml` | P/D disaggregation | template + stubs |
| `pd-cache-aware.yaml` | P/D + cache-aware hybrid | template + stubs + config.inline |
| `multi-pool.yaml` | Multi-pool VirtualService merge test | template + stubs |
| `moe.yaml` | MoE with DP/EP, 8 GPUs, RDMA/RoCE | template + stubs |
