# LLM-D Conformance Manifests — RHAIIS 3.4 EA1

LLMInferenceService manifests for RHAIIS 3.4 Dev Preview (EA1).

EPP scheduler config uses `scheduler.template.containers[].args` with `--config-text`.

Based on downstream samples: [red-hat-data-services/kserve (rhoai-3.4-ea.1)](https://github.com/red-hat-data-services/kserve/tree/rhoai-3.4-ea.1/docs/samples/llmisvc)

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
# Returns null → EA1 (this branch), returns object → EA2
```
