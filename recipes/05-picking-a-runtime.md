# Picking a serving runtime

**Question**: vLLM vs TGI vs llama.cpp vs Ollama — when does each one actually win?

## TL;DR matrix

| Runtime | Best fit | Avoid for |
|---|---|---|
| **vLLM** | High-concurrency, multi-tenant, A100/H100 fleets, OpenAI-compatible API needed | Edge / single-user / CPU-only |
| **TGI** | Hugging Face shops, model-card-driven workflows, streaming UX | Very large concurrency without serious tuning |
| **llama.cpp server** | CPU-only, low-end GPUs, quantised models, edge deployment | High-throughput multi-tenant SaaS |
| **Ollama** | Local dev, internal tools, "hey just try this model quickly" | Production serving above ~5 concurrent requests |

## How to actually choose

Decision factors, in order of importance:

1. **Hardware shape** — vLLM and TGI need real GPUs (preferably A/H100). llama.cpp and Ollama run on consumer GPUs and CPU. If you're CPU-only, the choice is made for you.
2. **Concurrency profile** — single-digit concurrent users → any of these. 100+ concurrent → vLLM (or vLLM-derived).
3. **Model format & license** — Ollama and llama.cpp need GGUF. vLLM and TGI use raw HF safetensors. Check that your target models exist in the format your runtime supports.
4. **API contract** — All four can speak OpenAI-compatible. Verify on your target version (TGI shipped this later than the others).
5. **Operational story** — vLLM has the deepest production tooling (Prometheus metrics, structured outputs, prefix caching). Ollama is the easiest to run. TGI sits in between.

## Don't switch for the wrong reasons

- "vLLM is 2× faster than TGI in benchmark X" — at your workload pattern, the gap is probably <20%
- "Ollama is easier to run" — true, but it's not designed for >1 concurrent user
- "llama.cpp is more efficient" — only true on CPU and low-end GPUs; vLLM dominates on A100s

## Benchmark, don't argue

Run [`pyxis3-ai/vllm-bench`](https://github.com/pyxis3-ai/vllm-bench) against each candidate with **your actual workload pattern** before committing.

## When to ignore this recipe

- If your CTO has already picked one and the cost of switching > 6 months of GPU spend, just live with the choice.
- If you're using a hosted API (OpenAI / Anthropic / Together / Bedrock), this recipe doesn't apply.
