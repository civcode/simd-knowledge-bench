# References and provenance

_Checked 2026-11-03. Links are included for traceability; inclusion does not imply that a result has been reproduced by this project._

## Primary knowledge source and target model

1. **SIMD Architecture Handbook**, civcode. Function-agnostic architectural principles, 24 chapters, 15 abstract case studies. [Repository](https://github.com/civcode/simd-architecture-handbook) · [Book](https://github.com/civcode/simd-architecture-handbook/tree/main/book). Pin a specific revision for experiments.
2. **Qwen3.8-Flash-Next**, Qwen Team/Alibaba (2026). Planned model under test, not yet benchmarked here. [Official repository](https://github.com/QwenLM/Qwen3.8-Flash-Next) · [Hugging Face model](https://huggingface.co/Qwen/Qwen3.8-Flash-Next). Record a checkpoint hash, inference runtime/version and quantization; do not equate this MoE model with a conventional 6B dense model.

## Research on code optimization and SIMD

3. Kim et al. (2025), **ECO: Enhanced Code Optimization via Performance-Aware Prompting for Code-LLMs**. [arXiv:2510.10517](https://arxiv.org/abs/2510.10517) · [Project and reported comparisons](https://github.com/suhyeon0123/ECO). Combines bottleneck diagnosis and retrieval of optimization instructions without retraining the generator. The 7B result in [Research rationale](research-rationale.md) is **Best@5** from the ECO project's evaluation, not this project's measurement.
4. Zhao et al. (2026), **Multi-Source and Cross-Scenario Strategy-Guided Code Optimization (MoST)**. [arXiv:2607.20353](https://arxiv.org/abs/2607.20353). Extracts reusable strategies from several sources and transfers them across optimization scenarios.
5. He et al. (2025), **SimdBench: Benchmarking Large Language Models for SIMD-Intrinsic Code Generation**. [arXiv:2507.15224](https://arxiv.org/abs/2507.15224). 136 tasks across five instruction-set families.
6. Li et al. (2026), **AutoVecCoder: Teaching LLMs to Generate Explicitly Vectorized Code**. [arXiv:2605.17978](https://arxiv.org/abs/2605.17978). Specialized 8B model trained using intrinsic-knowledge-augmented data and efficiency-based reinforcement learning; **not** a prompt-only handbook evaluation.

## Retrieval and context

7. Li et al. (2024), **Retrieval Augmented Generation or Long-Context LLMs? A Comprehensive Study and Hybrid Approach**. [EMNLP Industry](https://aclanthology.org/2024.emnlp-industry.66/). Long-context and RAG comparison; its tasks are not SIMD optimization.
8. Liu et al. (2024), **Lost in the Middle: How Language Models Use Long Contexts**. [TACL](https://aclanthology.org/2024.tacl-1.9/). Demonstrates that relevant information may be less usable depending on position in a long prompt.
9. Zhao et al. (2024), **LongRAG: A Dual-Perspective Retrieval-Augmented Generation Paradigm for Long-Context Question Answering**. [EMNLP](https://aclanthology.org/2024.emnlp-main.1259/). Supports studying coherent retrieval units, but does not establish outcomes for the SIMD handbook.

## Benchmark suites and source repositories

10. **PolyBench/C 4.2.1**, 30 numerical kernels. [Source mirror and README](https://github.com/MatthiasJReisinger/PolyBenchC-4.2.1) · [License](https://github.com/MatthiasJReisinger/PolyBenchC-4.2.1/blob/master/LICENSE.txt).
11. **TSVC-2: Test Suite for Vectorising Compilers**, updated C implementation. [Source](https://github.com/UoB-HPC/TSVC_2) · [License](https://github.com/UoB-HPC/TSVC_2/blob/master/license.txt).
12. **Auto-vectorization benchmark with objective withdrawal of information**, Sergi Siso et al. [Source](https://github.com/sergisiso/autovec-benchmark). Includes TSVC extensions and six application microkernels.
13. **SimdBench**, source archive referenced by its authors. [Archive](https://anonymous.4open.science/r/SimdBench-1B3F/). Verify current accessibility and redistribution conditions.
14. **VectorBench**, scalar and hand-vectorized implementations from several upstream projects. [Source](https://github.com/revec/VectorBench). Licensing varies by incorporated code; SPEC is not freely redistributable.
15. **PIE: Performance-Improving Code Edits**, general code-performance dataset. [Source](https://github.com/LearningOpt/pie). Not specifically a SIMD architecture benchmark.
16. **Pattern-Driven Optimization Benchmark**, C optimization-pattern workloads. [Source](https://github.com/wlu03/pattern-driven-optimization-benchmark). Treat counts, quality and license claims as requiring validation at the pinned revision.

## Interpretation notes

- A source's reported speedups, accuracy or correctness rates depend on its evaluation protocol and may be **incomparable** with other papers.
- RAG-versus-long-context findings from QA and retrieval are **motivation for ablations**, not evidence that RAG wins on SIMD.
- AutoVecCoder's specialized training results are not evidence that simply prompting a base model with a handbook yields the same improvements.
- Additional source-specific license notices and provenance should be recorded in each imported task's manifest.
- Literature and repositories change. Pin revision IDs and revisit unverified source/license details during implementation.

Start with [Research rationale](research-rationale.md), [Experiment design](experiment-design.md) and [Benchmark datasets](benchmark-datasets.md).
