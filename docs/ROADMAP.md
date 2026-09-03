# nano_llm Roadmap v0.1

## Phase 0 — Measurement first

Build reproducible baselines before adding architectural complexity.

- tiny dense decoder baseline
- deterministic tokenizer/data manifest
- tokens/sec, VRAM, RAM, optimizer memory, checkpoint size
- perplexity + code/math/Korean held-outs
- exact active/stored/resident parameter accounting

Promotion gate: baseline is reproducible across at least 3 runs and can train on the reference RTX 4070-class machine.

## Phase 1 — Detachable Weight Packs

Introduce a shared core and 4–8 independently loadable packs.

Measure:
- equal-active-compute quality versus dense baseline
- router entropy and collapse
- module specialization without semantic hardcoding
- pack removal ablations
- catastrophic interference when adding a new pack

Promotion gate: logical parameter count can increase without proportional active FLOPs, with no material quality regression.

## Phase 2 — Residency hierarchy

Implement HOT/WARM/COLD storage with instrumentation before attempting aggressive streaming.

- VRAM hot cache
- pinned/system-RAM warm cache
- NVMe cold store
- asynchronous prefetch
- transfer/compute overlap
- module use prediction

Measure p50/p95/p99 token latency, cache hit rate, bytes transferred/token, and throughput after cold misses.

Promotion gate: exceeding VRAM capacity causes graceful degradation rather than a latency cliff.

## Phase 3 — Multi-token generation

Train multi-token prediction heads and compare 1/2/4-token horizons.

Measure:
- accepted tokens per forward
- verification/rejection rate
- output quality
- total end-to-end tok/s
- extra training cost

Promotion gate: real wall-clock speedup at matched quality, not just more predicted tokens.

## Phase 4 — Memory runtime

Separate long-lived task state from raw context.

- exact local token window
- working memory
- episodic store
- semantic/consolidated store
- learned retrieval
- reversible provenance back to source spans where practical

Promotion gate: long-horizon tasks use less active context while preserving or improving capability.

## Phase 5 — Continual module growth

Train/add packs while keeping most stored weights frozen.

Measure:
- capability added per trainable parameter
- backward transfer/interference
- router adaptation
- compatibility across checkpoints
- consolidation/merge experiments

Promotion gate: the model can gain a new capability without full-model retraining and without large losses in old capabilities.

## Phase 6 — Larger logical model

Only after earlier gates pass:

- grow module count
- increase stored parameter budget
- test 10B+ logical size while keeping active budget bounded
- stress NVMe/RAM/VRAM streaming
- long-run training and recovery

The target is not a vanity parameter count. A larger logical model is promoted only if it provides measurable capability at bounded active compute.

## Core benchmark matrix

Every major architecture change is tested on:

- English general language
- Korean general language
- code generation/repair
- C/C++/CUDA reasoning
- mathematics
- long-document retrieval
- multi-step structured tasks
- module cold-start latency
- module churn under mixed workloads

Every report includes:

`stored_params, resident_params, active_params, trainable_params, VRAM_peak, RAM_peak, NVMe_read_MB_per_token, tok_per_sec, p95_token_ms, capability_scores`.

## Freeze policy

A phase is frozen only when:

- tests are deterministic enough to reproduce the conclusion
- capability regression is absent or explicitly justified
- performance gain is measured end-to-end
- no benchmark-only routing rule exists
- module/residency behavior is observable
- checkpoint + config + data manifest + code commit fully identify the run
