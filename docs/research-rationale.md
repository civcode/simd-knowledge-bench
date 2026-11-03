# Research rationale and existing evidence

_Research note — 2026-11-03. Claims below refer to cited work, not to experiments performed in this repository._

## Motivation

The target capability is **runtime-optimal execution architecture**, not simply syntactically vectorized code. Optimizing a nested loop may require changing the schedule of logical work, choosing a vector-lane axis, preserving register-resident partials, altering data layouts, balancing cache reuse against accumulator traffic, or rejecting an illegal loop interchange.

The [SIMD Architecture Handbook](https://github.com/civcode/simd-architecture-handbook/tree/main/book) codifies such concepts in function-, language- and ISA-agnostic terms. Particularly relevant chapters include dependence analysis (02), lane mapping (03), arbitrary loop depths (04), loop transformations (05), stationary architectures (06), hierarchical tiling (07), memory layout (08), performance models (20) and architecture search (21).

**Core hypothesis:** a capable but comparatively compute-efficient coding model can transfer external architectural knowledge into better implementations without changing its weights.

This should be tested, rather than assumed. Providing a principle does not guarantee that a model can recognize when it applies, establish its legality, or correctly implement it.

## Relevant research

### ECO — performance-aware prompting (2025)

[ECO](https://arxiv.org/abs/2510.10517) extracts reusable runtime-optimization instructions describing inefficiencies and causes. Its inference pipeline combines a symbolic bottleneck advisor and retrieval of related instructions, without fine-tuning the generator.

The authors' [project results](https://github.com/suhyeon0123/ECO) report the following **Best@5** values for Qwen2.5-Coder-7B on PIE. The metrics and evaluation environment are specific to that work:

| Prompting approach | Reported correctness | Reported speedup |
| --- | ---: | ---: |
| Instruction only | 68.12% | 1.44× |
| Conventional RAG | 64.51% | 2.51× |
| ECO (advisor + retrieval) | 74.24% | 3.26× |

**Interpretation:** performance-aware external instructions can help a smaller code model. The comparison also suggests that naive retrieval and diagnosis-guided retrieval are not interchangeable. **Not established:** that a static, code-free SIMD handbook produces the same gains.

### MoST — transferable optimization strategies (2026)

[MoST](https://arxiv.org/abs/2607.20353) derives optimization strategies from multiple sources (including documentation and textbooks) and transfers strategies across tasks/languages, turning them into static-analysis rules. It provides direct conceptual support for representing optimizations above the level of one source-code pattern.

**Limitations for this project:** the method involves strategy extraction/matching infrastructure, examples, and more than adding prose to one model prompt. Its reported performance should not be presented as handbook-only improvement.

### SimdBench and AutoVecCoder (2025–2026)

[SimdBench](https://arxiv.org/abs/2507.15224) contains 136 SIMD-intrinsic coding tasks spanning SSE, AVX, NEON, SVE and RVV. The paper finds that SIMD generation is harder for LLMs than scalar generation.

[AutoVecCoder](https://arxiv.org/abs/2605.17978) improves an 8B model on explicit SIMD generation using domain-specific data synthesis and performance-driven reinforcement learning. This supports the usefulness of domain knowledge, but its training-based gains **cannot be attributed to prompt-only retrieval**.

### Long context versus retrieval (2024)

[RAG vs. Long-Context LLMs](https://aclanthology.org/2024.emnlp-industry.66/) found that long-context access generally had a performance advantage when computational resources were ample, while RAG offered substantial cost savings on the study's tasks.

[Lost in the Middle](https://aclanthology.org/2024.tacl-1.9/) showed that retrieving information from long prompts is not uniformly reliable across positions. These studies concern information use, not SIMD optimization, and motivate a test rather than determine its outcome.

## What remains unproven

- Whether the **entire handbook**, as-is, improves Qwen3.8-Flash-Next on architectural optimization tasks.
- Whether **RAG** can select the needed principle before the model already understands the bottleneck.
- Whether a small **permanent conceptual scaffold** plus retrieval is better than either approach alone.
- Whether guidance improves **correctness**, or sometimes causes over-optimization and illegal transformations.
- Whether the handbook changes **architecture decisions**, not only prompt verbosity or the number of attempted revisions.
- Whether any gains remain after accounting for **token budget, tool budget, compilation, profiling and measurement noise**.

## Testable hypotheses

- **H1 — knowledge:** Full-handbook prompting increases correct-and-faster rate over a matched no-handbook baseline.
- **H2 — feedback:** Runtime measurements improve optimization quality even without the handbook.
- **H3 — interaction:** Handbook-guided agents benefit more from performance feedback, or reach the same speedup in fewer iterations.
- **H4 — retrieval efficiency:** Relevant retrieved sections can preserve full-context optimization quality while reducing inference/context costs.
- **H5 — retrieval diagnosis:** A task-aware architectural diagnosis improves section selection over code-similarity retrieval alone.
- **H6 — generalization:** Gains persist on held-out, structurally transformed workloads and do not depend solely on matching handbook case studies.

For study links and provenance, see [References](references.md).
