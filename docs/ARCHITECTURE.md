# nano_llm Architecture v0.1

## 1. Design principle

The engine separates three quantities that ordinary dense LLMs often couple:

1. **Stored parameters** — everything the model has learned and can potentially use.
2. **Resident parameters** — weights currently cached in VRAM or RAM.
3. **Active parameters** — weights participating in the present forward step.

The architecture should scale stored capability much faster than active compute.

## 2. Core components

### 2.1 Shared Core
A compact transformer-like or hybrid sequence core remains resident. It owns token/latent embeddings, normalization conventions, the shared representation space, and interfaces to memory and detachable modules.

Initial prototype target: 300M–500M parameters, later tunable downward or upward from evidence.

### 2.2 Weight Packs
Weight Packs are independently loadable learned modules. They are not required to map permanently to human semantic labels. A pack may capture a learned computation cluster, style, skill, domain mixture, or reusable transformation.

Each pack exposes a versioned compatibility manifest containing dimensions, supported insertion points, quantization/storage format, training lineage, and shared-core compatibility hash.

### 2.3 Learned Router
The router predicts a sparse active set from the current latent state, recent routing trajectory, memory state, and optional prefetch horizon. Routing is learned rather than implemented with semantic if/else rules.

The router emits both **use-now** scores and **prefetch-next** scores.

### 2.4 Residency Manager
Three logical tiers:

- HOT: VRAM-resident modules
- WARM: pinned/system-RAM cache
- COLD: NVMe module store

Eviction and promotion are based on measured utility, predicted reuse, transfer cost, size, and recency. Correctness must not depend on a module already being hot; residency affects latency only.

### 2.5 Multi-token Decoder
The model should be trained to predict several future tokens/latent steps from one shared hidden state. The runtime may accept a speculative span only when verification/confidence gates pass. The first prototype must compare 1-token and N-token training under equal compute before promotion.

### 2.6 Memory Runtime
Raw context is kept for exact local sequence information, but long-lived history is progressively represented as:

- working memory — active task state
- episodic memory — temporally grounded events/interactions
- semantic memory — consolidated reusable facts/relations

Memory retrieval produces bounded latent/token blocks. Compression cannot silently discard information that the evaluation suite shows is still required.

## 3. Forward path

```text
input tokens
   -> shared core prefix
   -> router
       -> active pack set
       -> predicted next-pack set
   -> residency/prefetch scheduler
   -> sparse pack execution
   -> shared core integration
   -> memory read/write gates
   -> multi-token prediction heads
   -> verification / token commit
```

## 4. Training path

Training is staged so an RTX 4070-class machine can validate architecture before scale.

- Stage A: dense small baseline.
- Stage B: shared core + a few detachable packs.
- Stage C: learned sparse routing with load balancing and reuse metrics.
- Stage D: selective pack training while most stored packs remain frozen.
- Stage E: multi-token objective.
- Stage F: tiered residency simulation and real NVMe/RAM/VRAM streaming.
- Stage G: persistent memory and context compression.
- Stage H: continual pack creation/consolidation experiments.

## 5. Performance invariants

- Logical model growth must not linearly increase per-token FLOPs.
- A cold-module miss may increase latency, but sustained throughput must recover through prediction/prefetch/cache reuse.
- Module transfer and compute should overlap whenever hardware permits.
- VRAM overflow must degrade latency gracefully instead of causing an abrupt architecture change.
- Active pack budget remains bounded and observable.
- Every optimization is compared against quality/capability baselines.

## 6. Capability invariants

- No semantic hardcoding to make benchmarks pass.
- No special-case routing by dataset name.
- No assumption that more modules means more intelligence.
- New packs must prove measurable capability gain or compression/efficiency gain.
- Removing a pack must have measurable and attributable effects.
- Router collapse, expert starvation, and universal-expert domination are explicit failure modes.

## 7. First hardware profile

Target reference machine:

- NVIDIA RTX 4070 12 GB
- 32 GB system RAM
- NVMe SSD

Prototype budgets are intentionally conservative. First proof should remain below roughly 1B active parameters and should support small experiments that can be trained repeatedly rather than a single irreproducible giant run.

## 8. What is borrowed from NetEngine

Only architectural philosophy is borrowed:

- stored structure is not the same as active structure
- bounded active working set
- persistence outside the immediate computation window
- local learned routing instead of global dense participation
- scaling must be measured as active compute versus stored structure

nano_llm remains a language-model research project and does not inherit NetEngine's neural-fragment semantics, body/sensorium APIs, survival/affect systems, or runtime contracts.
