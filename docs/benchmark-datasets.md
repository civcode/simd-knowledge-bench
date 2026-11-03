# Candidate benchmark datasets

_Research inventory, 2026-11-03. Inclusion is proposed; no dataset has yet been imported or run._

## Selection criteria

A suitable task should require choosing among **legal execution architectures**, not simply naming an intrinsic. Prefer kernels with nontrivial loop dependencies, different lane mappings, output/input stationary alternatives, tiling/reuse trade-offs, irregular access, or variable work lengths. Evaluate final implementations with independent tests and runtime measurements.

Distinct benchmark roles:

1. **Architecture-sensitive optimization:** nested loops, memory order, cache behavior, tiling, register state.
2. **Vectorization legality:** aliasing, loop-carried dependencies, reductions, irregular indexing.
3. **Explicit-intrinsic competence:** correct use of specific instruction-set APIs (report separately).
4. **Out-of-distribution generalization:** production-oriented code and transformed/novel kernels.

## Shortlist

| Dataset | Scope | Best use | Important limitation |
| --- | --- | --- | --- |
| [PolyBench/C 4.2.1](https://github.com/MatthiasJReisinger/PolyBenchC-4.2.1) | 30 numerical kernels | Main architecture-sensitive suite | Related kernels, known benchmarks, compiler may already optimize |
| [TSVC-2](https://github.com/UoB-HPC/TSVC_2) | Approximately 150 vectorization tests (commonly reported as 151) | Dependencies, legality, restructuring | Microkernels need strong correctness checks and timing controls |
| [AutoVec benchmark](https://github.com/sergisiso/autovec-benchmark) | TSVC extensions and six application microkernels | Runtime/compile-time information constraints | Overlap with TSVC; select nonduplicate tests |
| [SimdBench](https://arxiv.org/abs/2507.15224) | 136 LLM SIMD-intrinsic tasks across five ISAs | Intrinsics competence secondary track | Hardware/ISA constraints; upstream code availability/license need verification |
| [VectorBench](https://github.com/revec/VectorBench) | Over 200 hand-vectorized functions, many scalar equivalents | Real-world generalization | Per-source licenses, dependencies, ISA requirements |
| [PIE](https://github.com/LearningOpt/pie) | Large performance optimization dataset | Broader code optimization cross-check | Many tasks reward algorithm changes unrelated to SIMD |
| [Pattern-Driven Optimization Benchmark](https://github.com/wlu03/pattern-driven-optimization-benchmark) | Pattern-oriented C optimization workloads | Additional architectural pattern testing | Newer dataset; validate provenance, splits, and claims independently |

Do not merge these heterogeneous tracks into a single opaque “SIMD” score.

## Recommended pilot (target ~40 tasks)

A candidate mix, subject to screening and reproducibility:

- **12 PolyBench/C:** e.g. `gemm`, `2mm`, `3mm`, `doitgen`, `atax`, `bicg`, `jacobi-1d`, `jacobi-2d`, `heat-3d`, `syrk`, `gesummv`, `fdtd-2d`.
- **16 TSVC-2:** stratify by reduction, recurrence, dependence, indirect addressing, loop restructuring, control flow; verify the chosen kernel IDs in upstream source.
- **6 AutoVec microkernels:** binomial options, Black–Scholes, Mandelbrot, convolution, small matrix multiplication, stencil.
- **6 SimdBench tasks:** restrict to an ISA supported on the target CPU; replace with architectural pattern tasks if source/license/access proves unsuitable.

This is a **proposed sample**, not a published benchmark split or a claim that every item is immediately compatible with the harness.

## Dataset-specific notes

### PolyBench/C

PolyBench provides configurable problem sizes, input initialization, kernel timing instrumentation, code marking and output dumping. This makes it well suited to comparing loop schedules and hierarchical tiling. Use multiple sizes spanning cache-resident and memory-bound regimes. A transformed candidate must preserve its mathematical contract, including floating-point tolerance rules.

Primary source: [PolyBench/C README](https://github.com/MatthiasJReisinger/PolyBenchC-4.2.1/blob/master/README). The source has an [Ohio State University distribution license](https://github.com/MatthiasJReisinger/PolyBenchC-4.2.1/blob/master/LICENSE.txt) with notice/redistribution conditions.

### TSVC-2

An updated C implementation of the classic Test Suite for Vectorizing Compilers. The [README](https://github.com/UoB-HPC/TSVC_2) specifically warns about historical bugs, checksum interactions and accidentally optimized-away work. Use independently checked reference outputs; ensure no dead-code elimination and measure sufficiently long-running loops.

Check its [license](https://github.com/UoB-HPC/TSVC_2/blob/master/license.txt) and retain required notices.

### AutoVec

Extends TSVC and six microkernels with controls for whether loop bounds, index offsets, branch-condition parameters, alignment and non-aliasing information is available at compile time. This provides natural “same algorithm, different optimization constraints” pairs.

Source: [AutoVec README and paper reference](https://github.com/sergisiso/autovec-benchmark). Check its `license.txt` and upstream contributions before copying code.

### SimdBench

Tests explicit intrinsic generation for SSE, AVX, NEON, SVE and RISC-V RVV, mixing correctness and performance. Only use a consistent target ISA/hardware for runtime ranking; report syntax/API knowledge separately from high-level scheduling skill.

Paper: [SimdBench (2025)](https://arxiv.org/abs/2507.15224). Paper-listed source: [SimdBench archive](https://anonymous.4open.science/r/SimdBench-1B3F/). Confirm accessibility and redistribution terms before importing.

### VectorBench

Production-oriented vectorized functions from projects including Simd, x265 and FastPFor. Useful for external validity, but code carries original sources' dependencies and licenses. Some bundled *scalar* suites (e.g. SPEC) are commercial; do not assume all content may be redistributed.

Source: [VectorBench README](https://github.com/revec/VectorBench).

## Admission checklist

For each task, capture:

- Source URL, upstream commit/hash, source license and attribution.
- Reference implementation, input/output contract, permitted floating-point error and aliasing rules.
- Target ISA, compiler flags, input sizes and allowed code-edit boundaries.
- Trusted correctness oracle, random/adversarial cases and at least one hidden test set.
- Baseline performance under identical `-O3`/ISA settings.
- Evidence that there is measurable *architectural* headroom, or an explicit tag as a negative control.
- Task category labels (e.g. `loop-interchange`, `output-stationary`, `memory-layout`, `reduction`).
- Known risk of overlap with handbook examples or widely circulated optimization answers.
- Whether an improvement changes algorithmic complexity versus execution architecture.

## Leakage and split policy

Split **by semantic kernel family**, not by random input size or cosmetic variant. Do not tune retrieval, handbook excerpts, prompts or test-selection rules against hidden evaluation tasks. Keep per-task gold architectural labels hidden from the coding agent and only use them for retrieval diagnostics/evaluation.

Before redistributing third-party code in this repo, verify and preserve its actual upstream licensing; prefer adapters that fetch a pinned upstream revision when appropriate.

See [Evaluation protocol](evaluation-protocol.md).
