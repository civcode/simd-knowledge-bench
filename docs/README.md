# SIMD Knowledge Bench — research documentation

SIMD Knowledge Bench is a proposed reproducible evaluation of whether **external architectural knowledge** helps an AI coding agent produce **correct, faster C/C++ SIMD-oriented code**, and whether that knowledge is best delivered through a full document or retrieval.

The motivating observation: models differ in their ability to reason about the *execution architecture* of nested loops—iteration order, dependency legality, cache reuse, lane mapping, accumulator lifetime, and memory layout—not merely their ability to emit intrinsics.

The knowledge source is the independent [SIMD Architecture Handbook](https://github.com/civcode/simd-architecture-handbook), whose 24 chapters describe function-agnostic principles using abstract pseudocode.

> **Status (2026-11-03):** Research and experiment design. No benchmark results from this repository have been collected. Numbers in the literature review belong to the cited publications, not this project.

## Documents

| Document | Contents |
| --- | --- |
| [Research rationale](research-rationale.md) | Motivation, hypotheses, related research, boundaries of existing evidence |
| [Experiment design](experiment-design.md) | Original 2×2 design, expanded 4×2 knowledge/feedback matrix, controls and ablations |
| [Benchmark datasets](benchmark-datasets.md) | PolyBench/C, TSVC-2, SimdBench, AutoVec, VectorBench, PIE, candidate pilot tasks |
| [Retrieval design](retrieval-design.md) | Full-context, basic RAG, hybrid diagnosis-guided RAG, and oracle retrieval |
| [Evaluation protocol](evaluation-protocol.md) | Correctness, measured runtime, speedup, validity, uncertainty, cost, and reproducibility |
| [Implementation roadmap](implementation-roadmap.md) | Proposed repository components, task interface, milestones and safeguards |
| [References](references.md) | Primary research papers, dataset repositories and related reading |

## Central questions

1. Does providing the handbook improve **initial** correctness and runtime performance?
2. Does it help an agent **find better optimizations with fewer runtime measurements**?
3. Can **retrieved subsections** match or beat the full handbook at lower inference cost?
4. Does **diagnosis-guided retrieval** outperform plain similarity-based RAG?
5. Which architecture-sensitive task families benefit, and where does guidance cause regressions?

The principal model under consideration is [Qwen3.8-Flash-Next](https://github.com/QwenLM/Qwen3.8-Flash-Next). Record an immutable model revision, quantization and inference configuration in every run.

## Research discipline

- Correctness is mandatory; an invalid program is not a performance win.
- Separate *architectural optimization* from *ability to recall ISA intrinsic syntax*.
- Run final performance evaluations in an isolated harness, never from agent-reported timings.
- Freeze benchmark/task splits and handbook revision before the main evaluation.
- Distinguish published results, design hypotheses, and measurements produced by this project.
- Benchmark execution should use local tooling. No automatic GitHub Actions workflows are part of the design.
