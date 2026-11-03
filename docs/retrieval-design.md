# Handbook delivery and retrieval design

## Question

Can retrieval of selected principles from the [SIMD Architecture Handbook](https://github.com/civcode/simd-architecture-handbook) match or exceed giving the model all 24 chapters, while reducing context/prefill cost?

**Challenge:** a model that fails to recognize a bad loop schedule may also fail to ask for the chapter on loop schedules. This is the *retrieval bootstrap problem*: missing knowledge may be needed to formulate the retrieval query.

Evidence is suggestive but not decisive. [ECO](https://arxiv.org/abs/2510.10517) reports benefits from a symbolic bottleneck advisor coupled to retrieval; [RAG vs. Long-Context](https://aclanthology.org/2024.emnlp-industry.66/) finds long context can be stronger when adequately resourced; [Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/) warns that inclusion in a long context does not guarantee reliable use. None directly settles this exact handbook/target-model experiment.

## Conditions

### Full handbook (C/D)

- Supply a frozen, complete copy of the handbook in a consistent prompt location.
- Track actual input tokens and prefix-cache hits; source byte count is **not** token count.
- Preserve headings, assumptions, cross-references and chapter order.
- Ensure the agent uses the same problem description and correctness instructions as other arms.

### Basic RAG (E/F)

- Index the handbook by **semantic Markdown subsections**, preserving parent chapter, heading path, and adjacent caveats.
- A fixed hybrid keyword + embedding search is a sensible *candidate* method; start with BM25/lexical retrieval if a simpler baseline is desired.
- Create retrieval queries from the task description and relevant code, using a controlled, identical prompt for every task.
- Return source text (not generated summaries) with stable identifiers, revision hash, rank and score.
- Use a fixed retrieval token budget; allow additional retrieval calls under a fixed call budget if part of the condition.

### Hybrid diagnosis-guided RAG (G/H)

Always include a compact, fixed **architectural scaffold**, around 1–2k tokens as an initial experiment. It teaches the *questions* to consider, without code-specific answers:

1. Identify logical axes: independent problems, outputs, contributions/reductions and ordered state.
2. Describe dependence/aliasing legality and exact-versus-tolerant numerical constraints.
3. Estimate lane mapping and access stride, unique/reused bytes, live accumulators and register pressure.
4. Identify likely candidate schedule families rather than assuming the current innermost loop is optimal.
5. Retrieve targeted sections, including assumptions and failure regimes, before choosing an implementation.
6. Measure and revise only when feedback access is permitted.

A lightweight **architecture diagnosis** produces queries based on the identified bottleneck, such as `output-stationary register pressure`, `loop interchange dependency legality`, or `ragged lane occupancy`. Retrieval returns relevant source sections and possibly one linked cautionary section.

In runtime-feedback arms, a new measured bottleneck can trigger another retrieval (e.g. spills after enlarging a tile). All such actions must be logged.

### Oracle retrieval (small ablation)

Supply expert-chosen sections without revealing solution code or measured outcomes. Compare:
- Oracle vs. full handbook: can a compact, correctly selected subset suffice?
- Basic/hybrid vs. oracle: how much performance is lost to imperfect retrieval?
- Basic vs. hybrid: is the diagnosis step useful?

## Suggested indexing pipeline

1. Pin the handbook **commit SHA**; do not retrieve from a moving `main` branch during experiments.
2. Parse each `book/*.md` file by Markdown heading hierarchy.
3. Store `section_id`, path, heading trail, body, token count, chapter, source SHA and links to nearby sections.
4. Avoid arbitrary fixed-size 500-token splitting when it separates an optimization claim from legality conditions.
5. Preserve tables, abstract pseudocode, examples and explicit warnings when retrieving.
6. Deduplicate near-identical excerpts from `HANDBOOK.md` and `book/` — index **one canonical copy**.
7. Log every query, selection, score, retrieved text and token count; make all inputs replayable.

The initial handbook is a small, well-structured corpus; a sophisticated external vector database is **not a prerequisite** for testing this hypothesis.

## Retrieval evaluation

Measure **section recall@k** against independently annotated relevant sections, precision/context waste, token count, retrieval latency, and downstream correct-and-faster rate. Gold relevant-section annotations must not be visible to the agent in the automated RAG arms.

A model can retrieve relevant principles but still fail to apply them. Keep **retrieval quality** distinct from **application quality**.

## Avoid confounding the comparison

- The hybrid scaffold itself is added knowledge: report this explicitly, and optionally test a scaffold-only control.
- Match candidate counts, available tools, task prompt and final timing environment.
- Pre-register retrieval budgets, not just `top_k`; section lengths vary.
- Compare actual inference latency and resource use. Full-context prefix caching can reduce repeated prefill costs.
- Do not put benchmark solutions, hidden test inputs, hand-selected ideal implementations or performance rankings into the index.
- Distinguish failure to retrieve, failure to reason with retrieved text, and failure to implement legal/efficient code.

## Handbook sections likely to co-occur

| Architectural symptom | Possible sections to retrieve |
| --- | --- |
| Nested loop with poor stride | 02 dependencies; 04 loop depths; 05 transformations; 08 memory layout |
| Many outputs sharing one input | 03 lane mapping; 06 stationary schedules; 07 tiling |
| Long reduction chain | 02 legality; 03 lanes; 09 lane communication; 12 reductions; 20 cost models |
| Register spills after tiling | 07 hierarchical tiling; 20 performance models; 21 architecture search |
| Masked/ragged work | 03 lane mapping; 10 control flow; 11 irregular work |
| Pipeline dominated by packing | 08 data layout; 18 fusion/recomputation; 20 total cost |

This table is an **illustrative retrieval design aid**; it should not be used as a hidden solution label for tasks in a measured run.

See [Experiment design](experiment-design.md) and [Evaluation protocol](evaluation-protocol.md).
