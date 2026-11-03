# Open design decisions

**Project:** SIMD Knowledge Bench  
**Status:** Decision register — no items below are approved merely because a default is suggested  
**Revision date:** 2026-11-03  
**Companion:** [Architecture specification](architecture-specification.md)

This register separates **agreed research requirements** from **implementation choices not yet made**. Each item has a stable ID, a priority, a proposed default, alternatives, and a decision gate. Convert an item to `Accepted` only after the team explicitly agrees, recording the date, reason, and any consequences. Keep superseded choices in history rather than erasing them.

## Constraints already established (not open decisions)

- Study the effect of architectural handbook knowledge on correct, runtime-efficient C/C++ SIMD-oriented code.
- First target model: **Qwen3.8-Flash-Next**; exact weights and serving configuration are still to be selected.
- Compare no-handbook/full-handbook with and without agent-visible **runtime** feedback (A/B/C/D); evaluate retrieval arms E/F/G/H afterward.
- All conditions may receive correctness/compiler feedback; **only** B/D/F/H may see development-time performance measurements.
- Use the independently maintained [SIMD Architecture Handbook](https://github.com/civcode/simd-architecture-handbook) as a source, pinned per study.
- Use isolated, hidden final correctness and runtime evaluation; never let final scores feed back into the agent.
- Run locally and minimize GitHub Actions use; do not create/enable automatically triggered workflows.

These constraints are part of the current research plan. Changes that affect them should be documented as protocol amendments.

## Decision summary

`P0` blocks trustworthy harness/initial four-arm development. `P1` blocks the broader comparative pilot. `P2` can usually wait until the retrieval or scale-up phase.

| ID | Priority | Question | Suggested starting default | Gate |
| --- | --- | --- | --- | --- |
| D-01 | P0 | Implementation language / packaging | Python CLI orchestrator; C/C++ task code | Harness start |
| D-02 | P0 | Generated-code isolation | Restricted local container, network off | Before executing generated code |
| D-03 | P0 | Target CPU and supported ISA | One dedicated, pinned x86-64 host/ISA profile | Before first timed comparison |
| D-04 | P0 | Compilation policy | One pinned GCC/Clang profile; `-O3`, explicit target | Before task admission |
| D-05 | P0 | Model serving interface | Local OpenAI-compatible endpoint adapter | First agent run |
| D-06 | P0 | Candidate budget and stop rule | 5 candidates; equal cap across arms | Before A/B/C/D pilot |
| D-07 | P0 | Candidate selection rule | Deterministic final-valid for no-feedback; best visible score for feedback | Before A/B/C/D pilot |
| D-08 | P0 | Correctness oracle and tolerance | Per-task explicit equivalence contract | Before any score is valid |
| D-09 | P0 | Feedback benchmark vs. hidden scorer | Distinct data/config and unidirectional permissions | Before feedback runs |
| D-10 | P1 | Pilot datasets and task split | PolyBench/C + TSVC-2; split by semantic family | Before task suite freeze |
| D-11 | P1 | Timing methodology | Interleaved repeated medians; fixed environment | Before performance claims |
| D-12 | P1 | Results/artifact persistence | Append-only run directories + structured JSON/JSONL | First reproducible pilot |
| D-13 | P1 | Handbook composition | Canonical pinned `book/` source; full-book arm retains all chapters | Before handbook arms |
| D-14 | P1 | Prompt/tool parity | One shared prompt/tool contract with arm-specific policies | Before A/B/C/D pilot |
| D-15 | P1 | Statistical reporting and invalid submissions | Correct-and-faster primary; conditional geomean secondary | Before analysis freeze |
| D-16 | P1 | Model inference configuration | Pin checkpoint, quantization, temperature and seed policy | Before first agent run |
| D-17 | P1 | Task edit scope and packaging | Function/kernel-only edits where feasible | Before dataset adapters |
| D-18 | P2 | Basic retrieval implementation | Markdown heading sections, lexical BM25 baseline | Before E/F |
| D-19 | P2 | Hybrid diagnosis scaffold | Fixed ~1–2k token checklist + retrieval | Before G/H |
| D-20 | P2 | Retrieval limits and indexing | Fixed token and call budget; index one canonical book copy | Before E/F/G/H |
| D-21 | P2 | Oracle-retrieval annotation | Independent task-to-section labels for subset | Before RAG ablation |
| D-22 | P2 | Parallelism and scheduling | Sequential experiment runs initially | Scale-up |
| D-23 | P2 | Hardware generalization | Single-host results first; optional second architecture later | External-validity phase |
| D-24 | P2 | Artifact publishing and third-party licenses | Publish manifests/scripts; gate redistribution by license | Before public dataset release |

All 24 are **Open** at the time of writing. Suggested defaults are not final decisions.

## P0 — foundational decisions

### D-01 — Project language, CLI and dependency management

- **Options:** Python package with typed modules and CLI; Rust orchestrator; mixed services.
- **Recommendation:** Python for experiment orchestration/statistics and local CLI, with generated kernels and benchmark harnesses compiled in C/C++; modularize adapters so native backends can be changed.
- **Trade-off:** Python improves research iteration, but timing-sensitive loops must run outside the Python interpreter.
- **Decision test:** Can a two-kernel proof of concept be invoked, replayed and audited via the same public API?
- **Gate:** Before first implementation.

### D-02 — Executing untrusted generated code

- **Options:** Bare subprocess plus OS limits; rootless container with CPU/memory/filesystem/network restrictions; VM/microVM.
- **Recommendation:** Privilege-reduced local container with no network, read-only task inputs, no credentials and per-job time/memory limits.
- **Trade-off:** Isolation has operational requirements and can affect timing; correctness/compilation sandbox and stable timed executor may need different setups under the same threat model.
- **Decision test:** Intentionally malicious fixture cannot access host secrets, network or grader-only files; invalid code is terminated.
- **Gate:** **Before any model-generated native code is executed.**

### D-03 — Reference hardware and ISA

- **Options:** One dedicated x86-64 target with AVX2; AVX-512 capable host; ARM NEON/SVE; multi-ISA matrix.
- **Recommendation:** Start with **one machine, explicitly pinned ISA and CPU profile**; use AVX2 *if supported by that actual machine*. Do not infer hardware from the user's location or from model specifications.
- **Trade-off:** A narrow target makes reproducibility possible but limits portability claims.
- **Decision test:** Save CPU model, microcode, ISA availability, clock/governor policy, affinity and memory setup.
- **Gate:** Before accepting a performance task.

### D-04 — Compiler, flags and numerical semantics

- **Options:** GCC, Clang, or multiple; `-O3` with explicit target ISA vs. `-march=native`; permissive fast-math vs. strict floating-point semantics.
- **Recommendation:** Pin a single compiler + version and explicit production flags for the primary study; prefer strict semantics by default, with task-specific tolerances where justified.
- **Trade-off:** `-ffast-math` can change correctness and interact with vectorization; different compilers optimize differently.
- **Decision test:** Baseline and candidates build under identical flags, and oracle validates the semantics that will be benchmarked.
- **Gate:** Before dataset screening.

### D-05 — Local model runtime and agent protocol

- **Options:** vLLM; SGLang; llama.cpp if compatible; other local backend exposing a consistent API.
- **Recommendation:** Build a model-agnostic adapter around an OpenAI-compatible local endpoint when supported, and record backend/version and supported reasoning/tool-calling features.
- **Trade-off:** Differences in server defaults, sampling, context handling, caching and tool calls can alter the experiment.
- **Decision test:** Fixed test prompt, trace and output can be logged with exact token usage, sampling configuration and model fingerprint.
- **Gate:** Before first agent run.

### D-06 — Candidate count, feedback calls and stopping

- **Options:** Single-pass only; fixed 3/5/10 candidates; matched token budget; unlimited until time cap.
- **Recommendation:** Primary pilot: up to **5 generated candidates per task/arm/trial**, same correctness-test access; separately report candidate 1. Cap feedback measurements in eligible arms.
- **Trade-off:** Equal candidates do not imply equal inference cost, especially for the full handbook; a fixed budget simplifies causal interpretation.
- **Decision test:** Runtime policy actively refuses a sixth candidate/extra feedback call and logs the refusal.
- **Gate:** Before comparing A/B/C/D.

### D-07 — Selecting the final candidate

- **Options:** Last correct submission; earliest valid; model explicitly selects; lowest **agent-visible** timing in feedback arms.
- **Recommendation:** For A/C/E/G, deterministically use the **last valid** submitted candidate; for B/D/F/H, choose the fastest **validated** candidate from permitted *visible* feedback (predefine treatment of ties, missing measurements and noise).
- **Trade-off:** Different selectors reflect different information access; compare candidate-1 results separately, and record the selector itself as part of the arm definition.
- **Decision test:** Given a fixed trace, selection is deterministic and never consults hidden scorer results.
- **Gate:** Before pilot evaluation.

### D-08 — Correctness oracle and floating-point contract

- **Options:** Exact output comparison; abs/rel/ULP tolerance; reference algorithm plus randomized oracle; sanitizer-assisted validation.
- **Recommendation:** Each task declares its equivalence policy; validate across boundary cases, random inputs and additional hidden cases, and track sanitizer results independently.
- **Trade-off:** Incorrectly chosen tolerances can reward invalid reorderings or unfairly reject mathematically acceptable vectorization.
- **Decision test:** Deliberately incorrect, overflow-prone and out-of-bounds candidates fail the oracle.
- **Gate:** Before scoring any candidate.

### D-09 — Preventing final-score leakage

- **Options:** Single evaluator with separate methods; physically separated feedback/scoring processes and data; separate machine.
- **Recommendation:** Separate feedback and final scorer interfaces, distinct input sets and permissions; only the orchestrator can call the final scorer after candidate freeze.
- **Trade-off:** Stronger separation increases engineering work but protects the central claim.
- **Decision test:** A/C/E/G cannot obtain timings through public test APIs, returned fields, profiling tools or artifacts; all arms cannot read hidden scoring data.
- **Gate:** Before enabling runtime-feedback conditions.

## P1 — experiment validity and repeatability

### D-10 — Dataset subset, difficulty mix and train/test split

- **Options:** Pure PolyBench; PolyBench + TSVC-2; add AutoVec/SimdBench/production kernels.
- **Recommendation:** First harness proof: 2–3 manual kernels; next pilot: ~8–12 diverse PolyBench/TSVC kernels; then ~24–40 balanced tasks including negative controls.
- **Risk:** Popular kernels may be memorized; split by **semantic family**, not by input size or cosmetic transformations.
- **Gate:** Before benchmark suite freeze.

### D-11 — Timing protocol and reference implementation

- **Options:** Wall-clock medians; cycle counters; hardware counters; `Google Benchmark` or custom harness.
- **Recommendation:** Measure reference/candidate in randomized or interleaved order, warm up, retain raw repetitions and median/spread; profile hardware counters only diagnostically.
- **Open detail:** Sample duration, repetitions, outlier handling, power/frequency settings, data regime weighting and whether reference is original `-O3` code or a tuned scalar baseline.
- **Gate:** Before making speedup claims.

### D-12 — Storage and provenance

- **Options:** JSONL + content-addressed files; SQLite metadata + blobs; DuckDB/Parquet; remote database.
- **Recommendation:** Start with append-only files using stable run IDs and content hashes; choose SQLite only if querying/resume functionality makes it valuable.
- **Risk:** Storing large prompts and binary artifacts can be expensive; support retention policy without losing analysis provenance.
- **Gate:** Before a multi-arm reproducible pilot.

### D-13 — Handbook version and content policy

- **Options:** `HANDBOOK.md` single file; ordered `book/*.md` concatenation; selected chapters; principles-only document.
- **Recommendation:** Pin a handbook commit and use the ordered `book/` chapters as canonical; preserve all chapters including case studies in the **full-handbook** arm. Evaluate principles-only separately as an ablation.
- **Risk:** Case-study overlap can inflate apparent transfer; no moving `main` references during experiments.
- **Gate:** Before C/D.

### D-14 — Shared prompts and tool permissions

- **Options:** Uniform task instruction and varying injected context; freely different prompts per arm.
- **Recommendation:** Keep base system/task prompts, tool names and correctness policy equivalent; vary only authorized knowledge delivery, retrieval tools and timing observations.
- **Open detail:** Whether non-feedback arms can request an extra static-reasoning turn and whether benchmark-call denials are visible.
- **Gate:** Before A/B/C/D.

### D-15 — Primary outcome and uncertainty

- **Options:** Conditional geometric-mean speedup; correct-and-faster rate; failure-aware score; efficiency/cost curves.
- **Recommendation:** Primary = correctness plus **correct-and-faster at a predeclared threshold**; conditional geomean and sample-efficiency as secondary metrics. Report task-family level confidence intervals and all invalid attempts.
- **Open detail:** Threshold (10% proposed), failure penalty if any, bootstrap method, stratification and minimum detectable effect.
- **Gate:** Before collecting confirmatory results.

### D-16 — Exact Qwen configuration

- **Options:** Unquantized vs. quantized checkpoint; deterministic/low-temperature vs. stochastic seeds; different context limits, tool-calling and thinking modes.
- **Recommendation:** Pin a specific checkpoint and serving/quantization version with one predeclared sampling setup; store every raw response and agent transcript.
- **Risk:** Quantization, reasoning mode and endpoint semantics can dominate differences between knowledge conditions.
- **Gate:** Before first agent run.

### D-17 — Allowed code-edit boundary

- **Options:** Replace only the hot function; edit whole translation unit; change data layout/API; add helper files.
- **Recommendation:** Start with **kernel/function-scoped** edits and a stable harness API. Add layout/packing changes only when setup and end-to-end costs can be measured fairly.
- **Risk:** Too-restrictive edits exclude genuine architecture-level optimizations; too-broad edits enable benchmark gaming or unrelated algorithm changes.
- **Gate:** Before writing each task adapter.

## P2 — retrieval and scale

### D-18 — Retrieval engine

- **Options:** Exact chapter routing; BM25 lexical search; embedding vectors; hybrid rerank.
- **Recommendation:** Start with Markdown-heading-aware lexical retrieval, adding semantic retrieval only if recall/quality justifies the complexity.
- **Gate:** Before E/F.

### D-19 — Hybrid scaffold and diagnosis

- **Options:** No scaffold; fixed 1–2k-token architecture checklist; model-generated diagnosis; deterministic/static-analysis diagnosis.
- **Recommendation:** One versioned, compact universal checklist + explicit architecture diagnosis before retrieval; record its token cost and test scaffold-only if feasible.
- **Gate:** Before G/H.

### D-20 — Chunking and retrieval budget

- **Options:** Whole chapters; subsections plus caveats; arbitrary fixed-length chunks; top-k count vs. token-limited returns.
- **Recommendation:** Preserve Markdown sections/assumptions, retrieve under a **fixed token budget** with fixed maximum calls; index `book/` once and exclude duplicate `HANDBOOK.md`.
- **Open detail:** Token cap, cross-reference expansion, reranking, additional retrieval after agent-visible failures.
- **Gate:** Before E/F/G/H.

### D-21 — Oracle retrieval annotations

- **Options:** Hand label relevant sections; independently label by two reviewers; derive labels from expected transformations (risk of leakage).
- **Recommendation:** Annotate a modest held-out subset independently; retain ambiguity and do not expose labels to regular RAG models.
- **Gate:** Before oracle-retrieval ablation.

### D-22 — Experiment scheduling and parallelism

- **Options:** Sequential; task-parallel; concurrent agents but serialized timing; distributed workers.
- **Recommendation:** Sequential jobs and **exclusive timed measurements** initially; separate model inference concurrency from benchmark timing only after the harness is stable.
- **Risk:** Concurrent workloads can contaminate cache/clock measurements.
- **Gate:** Before scale-up.

### D-23 — Portability/generalization study

- **Options:** One CPU; second x86 implementation; ARM NEON/SVE; another model family; compiler cross-check.
- **Recommendation:** Primary conclusions on a single pinned target, then replicate a frozen subset across a second platform/model without tuning to the new results.
- **Gate:** External-validity study.

### D-24 — Publication, provenance and licenses

- **Options:** Vendor third-party sources into repo; pinned downloader/adapters; publish only manifests/reports; bundled snapshots.
- **Recommendation:** Pin URLs and commits; prefer adapters/downloaders and preserve upstream notices; publish derived artifacts only when permitted and strip secrets/private paths.
- **Gate:** Before importing/redistributing upstream benchmark code.

## Decision-making sequence

### Before coding

Resolve **D-01, D-02, D-03 and D-04**. These determine the test harness and whether performance results can be trusted.

### Before the first agent pilot

Resolve **D-05 through D-09**, **D-14**, **D-16** and **D-17**, alongside a provisional D-10 task set. These determine what the model is allowed to observe and how its final candidate is selected.

### Before publishing performance conclusions

Resolve **D-10 through D-13** and **D-15**; freeze the study version and document any deviations before running held-out tasks.

### Before retrieval experiments

Resolve **D-18 through D-21**. Otherwise a RAG-vs-full-context comparison mixes retrieval engineering differences with knowledge-delivery differences.

### Later

Resolve **D-22 through D-24** when scaling or releasing a dataset.

## Decision record template

When a choice is approved, update the corresponding entry and create a short decision note (ADR) if its consequences extend beyond one component:

```markdown
### D-XX — <title>
Status: Accepted | Superseded
Decided: YYYY-MM-DD
Decision: <concrete choice and version>
Alternatives considered: <options>
Rationale: <why this choice fits the research>
Consequences: <trade-offs, limitations, reproducibility impact>
Related study version(s): <IDs>
```

A change to a research-critical choice (e.g., selection rules, correctness policy, timing methodology, prompt/handbook revision or retriever budget) requires a new **study version**. Never retroactively relabel old runs as though the new choice had been in effect.
