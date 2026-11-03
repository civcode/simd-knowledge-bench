# Experimental design

## Research target

Evaluate whether architectural knowledge from the [SIMD Architecture Handbook](https://github.com/civcode/simd-architecture-handbook) improves **correctness and measured runtime performance** of generated C/C++ code, and whether document retrieval can replace full-context access.

Initial target model: [Qwen3.8-Flash-Next](https://github.com/QwenLM/Qwen3.8-Flash-Next) (record exact checkpoint and serving configuration). The model and benchmark runner are **not yet implemented or evaluated** in this repository.

## Phase 1: the original 2×2 factorial

Two independent factors: *handbook* (absent/present) and *runtime feedback during development* (absent/present).

| Arm | Handbook | Runtime feedback | Interpretation |
| --- | --- | --- | --- |
| A | None | No | Baseline |
| B | None | Yes | Effect of performance measurement |
| C | Full handbook | No | Effect of explicit architectural knowledge |
| D | Full handbook | Yes | Knowledge plus empirical optimization |

**Important:** all arms still have access to compilation errors and correctness testing. “Without benchmarks” means the **agent cannot see candidate timing results during optimization**; the final evaluator times all arms. For arms B/D, the agent can see runtime feedback and update the code.

For a strong matched-budget comparison, allocate the **same maximum candidate count and revision opportunities** to all arms. In A/C, allow revisions based on correctness/static reasoning; in B/D, also reveal timing measurements. Report candidate-1 performance separately to capture genuinely one-shot effects.

If this design also includes an unconstrained real-world-agent evaluation, label it as a separate, *pragmatic* experiment: unequal iteration budgets change the causal question.

## Phase 2: knowledge delivery × feedback (4×2)

| Knowledge mode | No runtime feedback | Runtime feedback |
| --- | --- | --- |
| None | A | B |
| Full handbook | C | D |
| Basic RAG | E | F |
| Hybrid RAG (diagnosis + persistent core) | G | H |

- **Basic RAG:** retrieve relevant handbook sections from source code/task description using a fixed retrieval policy and budget.
- **Hybrid RAG:** always include compact universal architectural concepts, diagnose the task's logical axes/dependencies/access patterns, retrieve relevant sections and linked caveats, then reason over candidates.
- **Full handbook:** use a frozen immutable source revision and a documented prompt order.
- **Runtime feedback:** the agent may measure candidates, inspect the results and update its implementation; all timing requests are logged.

The E/F versus C/D comparison measures whether retrieval preserves or improves full-handbook performance; G/H versus E/F isolates the value of diagnosis scaffolding.

## Additional ablations (subset)

1. **Principles only:** omit Chapters 23–24 (case studies, exercises) and optionally Chapter 21 (architecture-search procedure) to detect effects from answer-like examples or workflow coaching rather than transferable principles.
2. **Oracle retrieval:** evaluator-selected relevant sections supplied to the model, with no solution or performance leakage. This establishes an approximate upper bound on retrieval quality.
3. **Equal-token baseline:** provide an unrelated but task-neutral technical document or task-relevant controls of comparable length to distinguish useful knowledge from sheer prompt length, if feasible.
4. **Retriever recall:** audit whether the correct principles were supplied regardless of generated-code performance.
5. **Alternative model(s):** repeat selected conditions on another checkpoint after the primary protocol is frozen.

Ablations should be predeclared; exploratory runs must be labeled as such.

## Task and run allocation

- Start with a small **pilot** (approximately 24–40 source tasks) covering different architectural failure modes.
- Use 3+ independent seeds/trials per arm when generation is stochastic; distinguish tasks from repeated trials.
- Define and freeze a **held-out evaluation set** before prompt/retriever tuning.
- Ensure task variants sharing a semantic core remain within the same split to prevent leakage.
- Randomize the order of conditions across tasks to reduce environmental drift.
- Log the exact handbook commit, task revision, code changes, tool output, prompt messages, retrieval payloads and final executable hashes.
- Do not tune prompts or retrieval heuristics on the held-out test set.

## Control the agent budget

Recommended common budgets, chosen *before* testing:

| Resource | Rule |
| --- | --- |
| Candidate generations | Same maximum across all arms (e.g. 5) |
| Correctness tests | Same availability and limits |
| Compiler and flags | Same target ISA, compiler build, flags, libraries and environment |
| Runtime observations | Available only in B/D/F/H; fixed maximum number |
| Agent wall-clock / total tokens | Measure for all arms; optionally cap to comparable limits |
| Stopping criterion | Fixed, specified in advance; never select by hidden test timing |

Do not assume equal candidate counts imply equal *cost*: full handbook may have much higher token prefill. Report both matched-candidate and cost-aware results.

## Pre-registered questions and contrasts

For each task, let log-speedup be `y = log(T_reference / T_candidate)` for correct, valid implementations. For the 2×2 design:

- Knowledge effect without feedback: `C - A` in mean log-speedup.
- Feedback effect without knowledge: `B - A`.
- Interaction on log scale: `(D - C) - (B - A)`.

These formulas refer to **arm means on the log scale**, not raw individual runtimes. Handle invalid implementations separately (primary correctness and correct-and-faster metrics); do not silently exclude failed programs from performance conclusions. Report uncertainty with task-level or hierarchical bootstrap intervals.

The extended design adds contrasts for E/F/G/H. See [Evaluation protocol](evaluation-protocol.md) for metrics and aggregation.

## Threats to validity

- **Pretraining contamination:** PolyBench, TSVC, and popular SIMD benchmarks may already appear in model training; test transformed/unseen variants.
- **Handbook example overlap:** the book includes 15 abstract case studies and exercises; require novel held-out structures.
- **Algorithmic shortcuts:** restrict transformations to the stated semantic contract and explicitly report when a gain is algorithmic rather than architectural.
- **Compiler capability:** many scalar baselines already auto-vectorize at `-O3`; include these as negative controls, but stratify by available optimization headroom.
- **Timing noise and overfitting:** use hidden/independent measurement regimes for scoring.
- **Model nondeterminism and implementation drift:** pin versions, generation settings and seeds wherever possible.

See [Benchmark datasets](benchmark-datasets.md) and [Retrieval design](retrieval-design.md).
