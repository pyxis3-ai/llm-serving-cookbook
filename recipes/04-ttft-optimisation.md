# TTFT optimisation patterns

**Question**: Why is my time-to-first-token spiky and what do I tune?

## Where TTFT comes from

Time-to-first-token is the sum of:
1. **Queue wait** — request waited in vLLM's scheduler before being processed
2. **Prefill compute** — prompt encoded into KV cache (scales with prompt length)
3. **First-token decode** — one forward pass to emit the first token
4. **Network** — your gateway + LB + client round-trip

In production, 80%+ of TTFT spikes come from #1 (queue wait) and #2 (prefill on long prompts). The decode and network parts are usually fine.

## Tuning order

1. **Profile p50/p90/p99 separately.** TTFT distributions are heavy-tailed. p99 spikes have different causes than p50 drift.
2. **Cap your prompt length.** vLLM's `--max-model-len` clamps prefill cost. If your application doesn't need 128k context, don't pay for it.
3. **Enable prefix caching.** `--enable-prefix-caching` makes shared system prompts a zero-cost prefill. Most chat UIs benefit.
4. **Tune `--max-num-seqs` (batch size)** — too small and the GPU is idle; too large and queue wait climbs. Sweep this with `pyxis3-ai/vllm-bench`.
5. **Pre-warm with a synthetic request** on pod start so the first real request doesn't pay for CUDA graph compilation.

## What's usually NOT the problem

- Token-per-second rate (TPS) — affects TPOT, not TTFT
- Tensor parallelism config — only matters if you crossed the single-GPU memory boundary
- Sampling parameters (top-p, temperature) — sub-millisecond impact

## When to ignore this recipe

- If you need streaming UX and your TTFT p99 < 500ms, you're done. Don't over-tune.
- For batch inference workloads, TTFT doesn't matter. Maximise throughput instead.

## See also

- [`pyxis3-ai/vllm-bench`](https://github.com/pyxis3-ai/vllm-bench) — measure before tuning
