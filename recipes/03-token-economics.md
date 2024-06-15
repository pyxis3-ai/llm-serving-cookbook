# Token economics for multi-tenant serving

**Question**: How do I charge tenants fairly without making them hate the dashboard?

## The honest version

You're trying to recover GPU cost across many tenants. The naive approach is "per-token billing matching OpenAI rates". This works until a tenant's workload is bursty enough to crowd out other tenants, at which point your unit economics fall apart silently.

## What to actually measure

Three numbers per tenant per billing window:
1. **Prompt tokens consumed** — bills the input
2. **Completion tokens generated** — bills the output (usually 2-4× the input rate)
3. **GPU-seconds occupied** — covers the cold-start, model-load, and KV-cache-residency cost that pure token counts miss

The third one is the unlock. Per-token alone undercharges bursty workloads that hold KV cache without generating tokens (think: long-context retrieval-augmented queries).

## A workable scheme

- Charge **per-token** at a published rate close to upstream provider rates (so customers can sanity-check you)
- Charge a **separate "compute reservation" tier** for guaranteed throughput / dedicated capacity
- Track GPU-seconds internally but only expose it for tenants who exceed a fair-use threshold

## Implementation notes

- vLLM emits `vllm:request_inference_time_seconds_bucket` per request; tag with tenant ID via a header → metric label
- Use a write-optimised time-series store (Prometheus → ClickHouse export, or Timescale) — daily aggregation, not realtime
- Settle monthly, not daily — token counts have rounding noise at the day level

## When to ignore this recipe

- Single-tenant deployments don't need this — bill the customer in flat seats
- Free-tier SaaS doesn't need full GPU-seconds accounting — a simple monthly token cap is fine

## See also

- [`pyxis3-ai/pyxis-arch`](https://github.com/pyxis3-ai/pyxis-arch) — the multi-tenant control-plane thesis
