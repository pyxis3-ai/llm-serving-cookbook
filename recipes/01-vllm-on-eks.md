# Deploy vLLM on EKS

**Question**: How do I get vLLM serving a Llama 3 model on AWS EKS with proper GPU scheduling?

## The pattern

1. **Node group** — Use a dedicated EKS managed node group with `p4d.24xlarge` (8× A100) or `p5.48xlarge` (8× H100) instances. Taint with `nvidia.com/gpu=present:NoSchedule` so non-GPU workloads don't land here.
2. **NVIDIA device plugin** — Install the [NVIDIA k8s device plugin](https://github.com/NVIDIA/k8s-device-plugin) as a DaemonSet on the GPU node group. It exposes `nvidia.com/gpu` as a schedulable resource.
3. **Storage** — Use FSx for Lustre for the model weights cache. EBS gp3 is fine for boot but model loading dominates if you read from object storage on every pod start. Mount the FSx volume read-only across pods so weights are shared.
4. **vLLM Deployment** — One replica per GPU. Request `nvidia.com/gpu: 1` per replica. Tolerate the node taint. Set `--tensor-parallel-size 1` unless you genuinely need multi-GPU per request.

## Cost-aware tuning

- **Don't run multi-GPU tensor parallelism unless you need it.** A single A100 handles 70B-int8 fine; multi-GPU adds NCCL overhead and complicates scheduling.
- **Pin `--max-model-len`** to your actual prompt-completion budget. Default is the model max which over-allocates KV cache.
- **Use `--enable-prefix-caching`** if your workload has repeating system prompts (most do).

## When to ignore this recipe

- If your workload is bursty and brief, EKS Fargate isn't an option (no GPU support) but **AWS SageMaker async inference** or **Modal** may be cheaper than a hot EKS node.
- If you only have one customer, a single g6.xlarge with vLLM in Docker outside Kubernetes is dramatically simpler. Add K8s when you need it.

## See also

- [`pyxis3-ai/vllm-bench`](https://github.com/pyxis3-ai/vllm-bench) — measure your actual TTFT/TPOT after deployment
- [Recipe 02: Autoscale vLLM with KEDA](02-autoscale-keda.md)
