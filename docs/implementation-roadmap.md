# Implementation roadmap

This is a **proposed implementation plan**, not a claim that the benchmark runner exists. Build the minimum needed to answer the research question before expanding scope.

## Proposed project layout

```text
docs/                  # research notes and methodology (this folder)
configs/               # study manifests, model and machine configs
benchmarks/
  adapters/             # pinned upstream task ingestion, license notices
  manifests/            # task definitions, family labels and split metadata
  references/           # independent baseline/oracle where redistribution permits
src/
  agents/               # prompt & tool interfaces; no-feedback and feedback modes
  knowledge/            # full handbook, subsection index and retrieval modes
  execution/            # sandboxed compilation, correctness and timing
  evaluation/           # validity, aggregations, statistics and reports
tests/                  # fixture and harness tests
results/                # local output/summary schemas; raw results not necessarily tracked
```

Directories are placeholders. Do not copy third-party benchmarks into the repo before checking their licenses.

## Components

### 1. Task adapter

Standardizes each task's source code, contract, allowed edits, compiler/ISA, data types and input regimes. Provides a **trusted correctness oracle** and a reference baseline. Hidden final tests must be inaccessible to the agent.

Suggested task identity: `(dataset, upstream_sha, kernel_id, semantic_variant, target_isa)`.

### 2. Model/agent driver

Uses a pinned Qwen3.8-Flash-Next checkpoint (or another explicitly configured model), fixed sampling settings, maximum candidates and a replayable tool transcript. Supports both single-pass and multi-candidate optimization.

Agent interface should expose:
- source/task description;
- compiler diagnostics and permitted correctness feedback;
- handbook access according to the arm;
- **runtime measurements only when authorized by the experiment arm**;
- a consistent deadline/candidate budget.

Keep the task prompt identical across arms except for predeclared condition-specific knowledge and tool feedback.

### 3. Handbook loader

Pin a commit from [simd-architecture-handbook](https://github.com/civcode/simd-architecture-handbook). Offer:
- no knowledge;
- complete handbook;
- subsection retrieval;
- diagnosis-guided hybrid retrieval;
- oracle-selected sections for a limited ablation.

Index one canonical Markdown source tree, not both the single-file `HANDBOOK.md` and `book/` duplicates. Version all indexes and retrievers.

### 4. Sandboxed compiler and oracle

- Compile untrusted model-generated C/C++ inside controlled resource limits, ideally a container/VM with limited privileges and no network.
- Run correctness before timing; reject unsafe or invalid programs.
- Keep timed production builds separate from sanitizer/debug builds.
- Capture full source, compiler commands, stdout/stderr, exit status and executable hash.

### 5. Benchmark harness

Run reference and candidate binaries using controlled pinning, warmup and interleaved repetitions. Persist raw timing observations, not only the winning aggregate. Ensure the feedback benchmark cannot expose hidden final-scoring inputs.

### 6. Result aggregator

Produce a machine-readable record for each `task × arm × seed × candidate`, including validity, phase times, measured feedback, selected candidate, final hidden timings, token counts and retrieval logs. Provide task-level and dataset-family results plus uncertainty intervals.

## Suggested order of work

| Phase | Deliverable | Exit criteria |
| --- | --- | --- |
| 0. Documentation | Clear research questions, data sources, metrics | Reviewed protocol; frozen initial design |
| 1. Harness proof | 2–3 manual C/C++ kernels and independent correctness/timing | Repeated, stable local timings and rejection of wrong code |
| 2. Dataset pilot | ~8–12 diverse kernels from PolyBench/TSVC | Pinned sources, oracle, attribution and task manifests |
| 3. Four-arm experiment | A/B/C/D running reproducibly on one pinned model | Equivalent candidate budgets and independent final scoring |
| 4. Retrieval experiment | E/F/G/H with frozen section index | Traced queries, token budgets, retriever recall and ablations |
| 5. Expanded validation | ~24–40 pilot tasks; later held-out evaluation | Published raw results, uncertainty, failure analysis |

Freeze the runner and prompts before full evaluation. A smaller validated suite is preferable to a large suite with unreliable correctness or timing.

## Suggested manifest and result interfaces

A benchmark manifest should include:
- `task_id`, `family_id`, `source_url`, `source_sha`, `license`;
- `reference_path` / `oracle_id` and expected inputs/outputs;
- `compiler_flags`, `target_isa`, `input_regimes`, `tolerance_policy`;
- `task_description`, `editable_paths`, `timeout` and `memory_limit`.

A run should additionally capture:
- `model_id`, `model_sha`, `quantization`, `generation_settings`;
- `arm`, `handbook_sha`, `retriever_sha`, `scaffold_sha`;
- `candidate_index`, `generated_source_sha`, `compiler_output`, `oracle_result`;
- `agent_visible_timings`, `final_timings`, `input_tokens`, `output_tokens`, `elapsed_wall_time`.

## Safety and operating choices

- Run compilation and evaluation **locally** using installed command-line tooling.
- Do not create workflows that automatically start on `push`, `pull_request`, `schedule`, etc.
- Do not run or dispatch GitHub Actions without an explicit request. If CI workflows are later needed, make them **manual-only** (`workflow_dispatch`).
- Untrusted model-generated code must not access secrets or repository write credentials.
- Keep datasets/license notices separate from proprietary or restricted benchmark suites.
- Do not let hidden grading results enter model prompts or retrieval indexes.

See the [experiment design](experiment-design.md), [dataset inventory](benchmark-datasets.md) and [evaluation protocol](evaluation-protocol.md).
