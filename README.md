# LLM Serving Cookbook

Production recipes for running open-source LLMs on Kubernetes. vLLM-first, model-agnostic, K8s-native.

**Maintained by [PYXIS3](https://pyxis3.ai)**, model-agnostic LLM serving infrastructure.

---

## What's in here

These are recipes we've extracted from real production work. Each recipe is a self-contained explanation of *one* operational pattern, the kind of thing that takes weeks to get right the first time and 10 minutes the second time.

Recipes are markdown-only. Where code is needed, it's in `examples/` and is copy-paste-runnable.

| Recipe | What it answers |
|---|---|
| [Deploy vLLM on EKS](recipes/01-vllm-on-eks.md) | "How do I get vLLM serving a Llama 3 model on AWS EKS with proper GPU scheduling?" |
| [Autoscale vLLM with KEDA](recipes/02-autoscale-keda.md) | "How do I scale vLLM pods based on queue depth instead of CPU?" |
| [Token economics for multi-tenant serving](recipes/03-token-economics.md) | "How do I charge tenants fairly without making them hate the dashboard?" |
| [TTFT optimisation patterns](recipes/04-ttft-optimisation.md) | "Why is my time-to-first-token spiky and what do I tune?" |
| [Picking a serving runtime](recipes/05-picking-a-runtime.md) | "vLLM vs TGI vs llama.cpp vs Ollama, when does each one actually win?" |

---

## Conventions

- **Open-source first**: every recipe uses tooling you can self-host.
- **K8s-native**: assumes Kubernetes as the substrate, because production multi-tenant LLM serving doesn't run anywhere else.
- **Workload-realistic**: numbers come from real workloads, not benchmark loops.
- **Not a tutorial site**: we assume you can read a Helm chart and an AWS IAM policy. The recipes focus on the *decisions*, not the syntax.

## Why this exists

We kept seeing the same questions asked by teams running LLMs in production. The answers always existed, buried in vendor docs, GitHub issues, Slack threads, and people's heads. This is the consolidation.

Recipes are intentionally short and opinionated. If you disagree with one, open an issue with the workload pattern that contradicts it. We will either update the recipe or add a "when this doesn't apply" footnote.

---

## Contributing

PRs welcome. Format:
- One recipe per file in `recipes/`.
- Lead with the question, not the answer.
- Show the workload pattern that motivates the recipe.
- End with "When to ignore this recipe". Every pattern has limits.

---

## License

[Apache-2.0](LICENSE)

## Maintenance

Supporting documentation lives in `docs/`, example inputs live in `examples/`, and lightweight validation notes live in `tests/smoke/`.
