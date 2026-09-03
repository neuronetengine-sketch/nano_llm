# nano_llm

`nano_llm` is an experimental local-first language model project inspired by a subset of NetEngine's design philosophy:

> **stored capability != resident capability != active compute**

The goal is not to clone GPT/Claude or to wrap an existing hosted model. The project explores a new local LLM runtime/training path where a compact shared core can attach and detach many learned weight modules, keep only a bounded active working set in VRAM, spill cold modules through RAM/NVMe without catastrophic latency cliffs, and replace a monolithic token-history context with a persistent memory-oriented runtime.

## Primary goals

- Small always-resident **Core** that owns the shared representation space.
- Many detachable **Weight Packs** (adapters/experts) that can be mounted and evicted independently.
- **Predictive routing and prefetch** so likely next modules move toward VRAM before they are needed.
- **Tiered residency**: VRAM hot set, pinned/system RAM warm cache, NVMe cold store.
- **Multi-token generation** as a first-class training/inference objective rather than a late inference-only trick.
- Stable throughput when the logical model is larger than VRAM.
- Persistent **working / episodic / semantic memory** so useful history does not have to remain as raw tokens forever.
- Training that can grow the logical model by adding modules without retraining every stored parameter each time.
- Local hardware as the target, beginning with an RTX 4070 12 GB + 32 GB RAM class machine.

## Non-goals

- Semantic hard-coded experts such as `if programming -> load C++ pack` as the final routing mechanism.
- Benchmark-only routing rules.
- Hiding a remote proprietary model behind a local API.
- Claiming that a larger logical parameter count automatically means greater capability.

## Initial research questions

1. Can a shared core + detachable learned weight packs outperform an equally expensive dense baseline at the same *active* parameter budget?
2. Can expert/module prediction happen early enough that NVMe/RAM residency changes do not create large token-time spikes?
3. Can multi-token prediction reduce autoregressive synchronization cost without reducing output quality?
4. Can long-context dependence be replaced progressively by learned persistent memory while preserving exact local context when needed?
5. Can new capability packs be trained independently while retaining compatibility with the shared latent space?

## Repository plan

- `docs/ARCHITECTURE.md` — architecture and invariants
- `docs/TRAINING_DATA.md` — broad data curriculum and provenance policy
- `docs/ROADMAP.md` — staged experiments and promotion gates
- `nano_llm/` — prototype runtime/training package
- `configs/` — hardware and experiment configurations

The project deliberately starts small. The first milestone is not a 50B model; it is a reproducible proof that **logical scale can grow faster than active compute** without destroying quality or throughput.