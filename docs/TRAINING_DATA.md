# Training Data Strategy v0.1

The project needs broad data, but **dataset size is not the optimization target**. The target is a curriculum that teaches general language, reasoning, code, mathematics, multilingual use, long-document state tracking, and tool-like structured behavior while preserving provenance and allowing repeated experiments on local hardware.

## 1. Dataset tiers

### Tier A — General language / world knowledge
Use large, filtered web corpora as the backbone. Initial candidate: Hugging Face FineWeb-Edu / FineWeb-family samples. Full-scale corpora are too large for early local experiments, so training manifests must support deterministic streaming and percentage/token-budget sampling without copying the entire source locally.

### Tier B — Code
Use permissively usable, provenance-aware code corpora. Initial candidate: BigCode The Stack v2 / deduplicated variants, filtered by license and language. Early experiments should emphasize Python, C, C++, CUDA, Rust, JavaScript/TypeScript, shell, build files, and documentation rather than trying to sample every language equally.

### Tier C — Mathematics / formal reasoning
Initial candidate: FineMath-family datasets plus carefully selected open mathematical text and problem/solution material. Preserve equations and structured derivations. Do not convert every sample into chat format.

### Tier D — Scientific / technical prose
Open papers, documentation, manuals, standards-compatible excerpts, and educational material where redistribution/training terms permit it. The goal is dense explanatory language and cross-domain reasoning rather than trivia density.

### Tier E — Multilingual
Korean and English are first-class targets; broader multilingual data is desirable after pipeline validation. Korean data must not be treated as a translation-only adapter. The shared core should see substantial native Korean during pretraining, with later modules allowed to specialize further.

### Tier F — Long-document / memory curriculum
Books or long-form open texts, source trees, documentation sets, multi-file synthetic tasks, and long conversational/task traces. Samples preserve document boundaries and timestamps/ordering metadata so memory experiments can distinguish sequence time from host execution time.

### Tier G — Instruction / agentic behavior
Later-stage supervised data for planning, tool schemas, code editing, verification, self-correction, structured output, and multi-step task completion. This is post-pretraining curriculum, not a replacement for broad language modeling.

## 2. Initial mixture philosophy

Do not freeze percentages yet. Begin with a measurement-driven mixture roughly covering:

- broad natural language / educational web
- code
- mathematics and formal reasoning
- technical/scientific prose
- Korean + multilingual native text
- long-document sequences

Mixture weights are promoted only after ablations. Every mixture experiment records source tokens, effective sampled tokens, dedup rate, language distribution, sequence-length distribution, and held-out capability results.

## 3. Data volume strategy

The storage universe may be very large while a local training run consumes a bounded deterministic token stream.

Three scales:

- **Smoke**: 10M–100M tokens — architecture correctness and overfit checks.
- **Research**: 0.5B–5B tokens — repeated architecture comparisons on local hardware.
- **Scale**: 10B+ tokens — only after the architecture shows clear compute-efficiency gains; may require long-running local training or rented compute while keeping the runtime local-first.

A manifest defines the logical dataset independently from the amount physically cached on disk.

## 4. Streaming and storage

```text
remote/open dataset shards
        -> manifest + provenance filter
        -> deterministic sample stream
        -> local compressed shard cache
        -> tokenizer/packer workers
        -> training batches
```

Requirements:

- resumable downloads
- content hashes
- deterministic seed + shard order
- per-source token accounting
- duplicate / near-duplicate filtering
- held-out contamination checks for evaluation sets
- source and license metadata retained through preprocessing
- no requirement to keep the full corpus on the workstation

## 5. Module-oriented curriculum

Detachable Weight Packs are not assigned permanent human labels by default. Data can nevertheless be grouped into *training curricula* to test whether modules specialize naturally.

A module-training run records:

- shared-core checkpoint
- source mixture
- router state
- module initialization lineage
- trainable parameter count
- active parameter count
- optimizer state size
- capability gained/lost
- interference with existing modules

A newly trained pack is accepted only if it provides measurable benefit versus an equal-active-compute baseline.

## 6. Multi-token objective data

Every packed sequence can provide several next-token targets at each eligible position. The trainer should support 1, 2, 4, and later adaptive prediction horizons. Evaluation must measure both accepted tokens per forward pass and quality/perplexity effects; a nominal 4-token head that is usually rejected is not a speedup.

## 7. Korean strategy

Korean should exist at multiple levels:

1. native Korean in shared-core pretraining,
2. Korean-heavy curriculum for robust morphology/style coverage,
3. optional detachable packs if specialization proves useful,
4. Korean reasoning/code/document tasks in evaluation.

Do not make Korean capability depend entirely on one detachable language pack; removing one pack must not make the core unable to parse ordinary Korean.

## 8. Data governance

Before a source enters a release-grade manifest:

- record original dataset/project name and version
- record license / terms
- preserve source identifiers when available
- exclude sources whose use cannot be justified for the intended distribution/training setup
- make deletion/rebuild possible by source ID

Early private experiments may use separate manifests, but they must not silently become release manifests.

## 9. Initial candidate sources

Candidates to evaluate, not automatic blanket approval:

- HuggingFaceFW FineWeb-Edu / FineWeb family — broad filtered educational/web text
- BigCode The Stack v2 deduplicated family — code with source/license metadata workflow
- HuggingFaceTB FineMath family — mathematical text/reasoning
- additional Korean, multilingual, scientific, long-document, and instruction corpora selected after provenance/license audit

The architecture must never depend on a single provider or dataset naming scheme. Dataset adapters convert external formats into nano_llm's internal manifest/record format.
