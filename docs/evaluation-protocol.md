# Evaluation protocol and reproducibility

_No project benchmark results exist yet. This document specifies how results should be produced._

## Primary outcomes

1. **Correctness rate:** proportion of submitted implementations passing the complete independent oracle.
2. **Correct-and-faster rate:** proportion of *all tasks/trials* with a correct implementation at least 10% faster than the optimized scalar/reference baseline, under a predeclared threshold.
3. **Valid-code speedup distribution:** per-task `S = T_reference / T_candidate`, conditional on correctness; always display validity alongside this metric.
4. **Compute/search efficiency:** number of candidates, benchmark invocations, generated/prefill tokens, actual inference wall-clock time and time to target speedup.

Do not present a geomean over only successful optimizations as the sole outcome: removing failures introduces survivorship bias. In addition to conditional speedups, report a predeclared *failure-aware* score (e.g., assigning speedup zero to invalid submissions for a utility metric) and its sensitivity to the chosen penalty.

## Correctness

A candidate must:
- Compile under the pinned compiler/target ISA, without undefined behavior or unsafe overreads.
- Preserve functional contract, dimensional bounds, error handling, output shape and no-alias assumptions.
- Pass reference comparisons on random and adversarial inputs, multiple shape regimes and tail lengths.
- Respect numerical equivalence class: bit-exact, integer-defined, or explicit abs/rel/ULP floating-point tolerance.
- Preserve intended behavior for NaN/Inf, signed zero, overflow, saturation and stable ties when observable.

Use ASan/UBSan where feasible in a **separate validation build**, not in timed runs. Test aliasing/in-place and near-boundary sizes when relevant. Keep hidden correctness inputs outside the agent workspace.

## Timing setup

- Fix machine model, CPU stepping/microcode, supported SIMD ISA, OS/kernel, governor/boost policy, SMT, core affinity, memory configuration and compiler version.
- Compile baseline and candidate with **the same** production optimization flags (illustratively `-O3 -march=native` on a single pinned machine; record exact flags and treat `-ffast-math` as a change of semantics unless explicitly permitted).
- Benchmark meaningful work, prevent compiler elimination of results and exclude process startup/compilation unless the task specifically measures end-to-end latency.
- Include setup/packing/unpacking/dispatch costs whenever the proposed transformation requires them in production.
- Warm up appropriately; randomize/interleave benchmark order to mitigate thermal/frequency drift.
- Use repeated independent runs; record raw timings, median, spread and outliers rather than one best-case value.
- Test small, medium and large workloads spanning cache levels where feasible, plus irregular/adversarial distributions.
- Make agent-visible feedback measurements and **final hidden scoring measurements** separate.
- Use CPU counters and compiler vectorization/assembly reports as *diagnostics*, not primary speedup proof.

Useful compiler evidence: Clang's loop-vectorization remarks (`-Rpass=loop-vectorize`, `-Rpass-missed=loop-vectorize`).

## Speedup aggregation

For correct implementations on a predeclared common set of tasks, use a geometric mean:

```text
S_i = median(reference_time_i) / median(candidate_time_i)
geomean_speedup = exp(mean_i(log(S_i)))
```

For failures, separately publish success-conditioned geomeans and correct-and-faster rates with the **full denominator**. A score combining them requires an explicit failure penalty and is not directly comparable to a success-conditioned geomean.

If multiple input sizes exist for one semantic kernel, aggregate within kernel first, then across kernels; do not inflate statistical sample size by counting correlated sizes as independent problems.

## Experimental comparison

For original arms A/B/C/D, model the **task-level log-speedup** with two factors (knowledge and runtime feedback), optionally using a mixed-effects/hierarchical model to account for repeated samples and kernel families.

The multiplicative interaction (when meaningful and all speedups are defined) is:

```text
interaction_ratio = (S_D / S_C) / (S_B / S_A)
                  = (S_A * S_D) / (S_B * S_C)
```

Use paired task-level bootstrap confidence intervals or a hierarchical model. Also analyze binary correct-and-faster outcomes so invalid submissions are included.

For RAG, report the same contrasts as full handbook, with token/prefill/latency and retrieval-recall differences. Stratify by task category and CPU working-set regime; identify regressions as prominently as wins.

## Candidate selection and feedback leakage

- Pick a **selection rule** before evaluation (e.g. latest valid candidate, or best candidate under agent-visible measurements for feedback arms).
- If no-feedback arms cannot use timing to select from multiple valid candidates, do **not** secretly select their best by test performance and compare that with agent-selected candidates as though budgets were equal.
- Evaluate a fixed selected artifact with a hidden measurement harness; never let the agent edit code based on the final scoring set.
- Store every attempted candidate and feedback observation. Report candidate-1 results independently.
- Cap all agents equally by candidate count; state any additional wall-clock/token caps.
- Avoid rerunning until favorable results appear. Record all attempts and failures.

## Minimal run manifest (example; illustrative values)

```yaml
study_id: pilot-v1
task_id: polybench-gemm
task_revision: <pin-source-sha>
handbook_revision: <pin-handbook-sha>
model: Qwen3.8-Flash-Next
model_revision: <pin-checkpoint>
quantization: <record-exact-setting>
arm: D                      # A-H
generation_seed: 1234
max_candidates: 5
runtime_feedback_budget: 4
compiler: <name-and-version>
flags: "-O3 -march=native"
cpu: <model-and-microcode>
thread_affinity: <core-list>
oracle_revision: <hash>
retriever_revision: null
```

Keep machine-readable manifests and raw timing outputs as immutable artifacts in addition to summarized tables.

## Do not claim

- That faster generated code proves it is *globally optimal*; a finite benchmark can only establish relative measured performance.
- That an optimization is portable to other CPUs, compilers or input distributions without further testing.
- That fewer input tokens imply lower cost unless actual prefill/cache/latency data support it.
- That RAG “understands” architecture based only on retrieved-section overlap.

See [Experiment design](experiment-design.md) for arms and [Benchmark datasets](benchmark-datasets.md) for task screening.
