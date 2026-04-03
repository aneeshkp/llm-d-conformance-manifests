# LLM-D Conformance Manifests

LLMInferenceService manifests for [llm-d-conformance-test](https://github.com/aneeshkp/llm-d-conformance-test).

Each branch corresponds to a RHAIIS release. **Use the branch that matches your cluster's CRD version.**

## Branches

| Branch | Platform | Status |
|--------|----------|--------|
| `3.4-ea1` | RHAIIS 3.4 Dev Preview (EA1) | Active |
| `3.4-ea2` | RHAIIS 3.4 EA2 | Active |
| `main` | Latest (auto-synced from latest release branch) | Protected |

## Usage

```bash
# In the test framework repo:
make setup MANIFEST_REF=3.4-ea1    # EA1 cluster
make setup MANIFEST_REF=3.4-ea2    # EA2 cluster
```

## EA1 vs EA2: What Changed

| Feature | EA1 (`3.4-ea1`) | EA2 (`3.4-ea2`) |
|---------|----------------|----------------|
| **EPP scheduler config** | `scheduler.template.containers[].args` with `--config-text` (verbose) | `scheduler.config.inline` or `scheduler: {}` (CRD handles defaults) |
| **Cache-aware routing** | Explicit `--config-text` with `precise-prefix-cache-scorer`, `load-aware-scorer`, ZMQ config | `scheduler.config.inline` with same plugins, less boilerplate |
| **P/D routing** | Full `--config-text` with P/D scorer plugins | `scheduler: {}` (default scheduler handles P/D) |
| **Hash algorithm** | `sha256_cbor` | `sha256_cbor` |
| **CRD field** | `scheduler.template` only | `scheduler.config.inline` added |

### How to check your cluster version

```bash
# If this field exists in the CRD, you have EA2:
kubectl get crd llminferenceservices.serving.kserve.io -o json | \
  jq '.spec.versions[].schema.openAPIV3Schema.properties.spec.properties.router.properties.scheduler.properties.config'
# Returns null → EA1, returns object → EA2
```

## Manifests

| Manifest | Description |
|----------|-------------|
| `single-gpu.yaml` | 1 GPU with scheduler |
| `single-gpu-smoke.yaml` | 1 GPU, minimal smoke test |
| `single-gpu-no-scheduler.yaml` | 1 GPU, K8s native routing (no EPP) |
| `cache-aware.yaml` | Prefix KV cache-aware routing, 2 replicas |
| `pd.yaml` | P/D disaggregation (TCP) |
| `moe.yaml` | MoE with DP/EP, 8 GPUs, RDMA/RoCE |

## Contributing

1. Create or edit manifests on the appropriate release branch (e.g., `3.4-ea2`)
2. Push to the branch — `main` auto-syncs from the latest release branch
3. Do **not** push directly to `main` (it is locked)

Based on downstream samples: [red-hat-data-services/kserve (rhoai-3.4-ea.1)](https://github.com/red-hat-data-services/kserve/tree/rhoai-3.4-ea.1/docs/samples/llmisvc)
