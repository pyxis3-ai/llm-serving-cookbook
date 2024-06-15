# Autoscale vLLM with KEDA

**Question**: How do I scale vLLM pods based on queue depth instead of CPU?

## Why not HPA on CPU

vLLM saturates a GPU long before CPU does. By the time CPU crosses a threshold, you're already dropping requests. You need a signal that tracks **work-in-flight**, not resource utilisation.

## The pattern

Use [KEDA](https://keda.sh) (Kubernetes Event-Driven Autoscaling) with a Prometheus scaler pointing at vLLM's exposed metrics.

### Metrics that matter

vLLM exposes (via the OpenAI metrics endpoint or a Prometheus-side scrape):
- `vllm:num_requests_running` — current in-flight requests
- `vllm:num_requests_waiting` — queued requests not yet running
- `vllm:gpu_cache_usage_perc` — KV cache utilisation

### KEDA ScaledObject

Scale up when **waiting + running per pod > threshold** for 60 seconds. Conservative defaults:
- Threshold per pod: **4 active+queued requests** (tune to your model + GPU)
- Cool-down: **5 minutes** scale-down delay (model load is expensive)
- Min replicas: **1** (cold-start sucks for chat UIs; warm baseline is worth the cost)
- Max replicas: cap at your GPU node-group's actual capacity

### Cold start

Model loading is ~30–90s for 7B models, more for larger. Pre-pull the model image to nodes via [`kube-fledged`](https://github.com/senthilrch/kube-fledged) or bake it into the image. Without that, scale-up events are slow enough to be useless.

## When to ignore this recipe

- If you have steady predictable traffic, just pin replicas. Autoscaling on its own is operationally expensive.
- If you only have one model, the simpler `vllm:gpu_cache_usage_perc > 80%` is sometimes enough.

## See also

- [Recipe 03: Token economics for multi-tenant serving](03-token-economics.md)
