# LLM-D Conformance Manifests — RHAIIS 3.4 EA2

LLMInferenceService manifests for RHAIIS 3.4 EA2.

## What's New in EA2

- **Simplified scheduler config** — `scheduler.config.inline` replaces verbose `scheduler.template.containers[].args` with `--config-text`
- **P/D routing** — `scheduler: {}` is sufficient, CRD handles defaults
- **Cache-aware routing** — same plugins (`precise-prefix-cache-scorer`, `load-aware-scorer`) but configured via `scheduler.config.inline` with less boilerplate
- **New CRD field** — `scheduler.config.inline` added to the LLMInferenceService CRD

## Manifests

| Manifest | Description |
|----------|-------------|
| `single-gpu.yaml` | 1 GPU with scheduler |
| `single-gpu-smoke.yaml` | 1 GPU, minimal smoke test |
| `single-gpu-no-scheduler.yaml` | 1 GPU, K8s native routing (no EPP) |
| `cache-aware.yaml` | Prefix KV cache-aware routing, 2 replicas |
| `pd.yaml` | P/D disaggregation |
| `moe.yaml` | MoE with DP/EP, 8 GPUs, RDMA/RoCE |

## How to check your cluster version

```bash
kubectl get crd llminferenceservices.serving.kserve.io -o json | \
  jq '.spec.versions[].schema.openAPIV3Schema.properties.spec.properties.router.properties.scheduler.properties.config'
# Returns null → EA1 (use 3.4-ea1 branch), returns object → EA2 (this branch)
```
