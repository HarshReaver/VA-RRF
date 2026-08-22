# VA-RRF Literature Review: Bibliography

This document is the **bibliographic index** for the literature used to develop and position Variance-Aware Reciprocal Rank Fusion (VA-RRF).

Detailed paper-by-paper analysis is maintained separately in `Notes/literature_analysis.md`. The bibliography therefore focuses on metadata, classification, and the role each paper plays in the research rather than repeating the analytical discussion.

## Papers

| ID | Paper | Authors | Year | Venue | Link / Identifier | Role in VA-RRF |
|---:|---|---|---:|---|---|---|
| 1 | **Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods** | Cormack, G. V., et al. | 2009 | SIGIR | [DOI: 10.1145/1571941.1572114](https://doi.org/10.1145/1571941.1572114) | Foundational RRF paper; establishes the original RRF formulation and the use of the $k$ constant. |
| 2 | **An Analysis of Fusion Functions for Hybrid Retrieval** | Bruch, S., et al. | 2023 | ACM TOIS | [arXiv:2210.11934](https://arxiv.org/abs/2210.11934) | Central evidence for comparing RRF with Convex Combination and studying fusion-parameter sensitivity. |
| 3 | **RAG-Fusion: A New Take on Retrieval-Augmented Generation** | Rackauckas, M. | 2024 | Preprint | [arXiv:2402.03367](https://arxiv.org/abs/2402.03367) | Shows the use of RRF for combining results from multiple generated queries in RAG. |
| 4 | **Weighted Reciprocal Rank Fusion RAG for Context-Aware DoS Attack Mitigation** | Kafi, A., Saha, S. | 2026 | IEEE CCNC | [dblp:KafiSS26](https://dblp.org/rec/conf/ccnc/KafiSS26.html) | Contemporary work using weighted RRF; relevant as competing adaptive RRF work. |
| 5 | **Comparative Evaluation of Rank and Score Fusion Methods for Hybrid Search** | Cormack, G., Clarke, C., et al. | 2025 | SIGIR | [DOI: 10.1145/1571941.1572114](https://dl.acm.org/doi/10.1145/1571941.1572114) | Relevant comparison of score-based fusion and RRF, particularly the trade-off between score information and rank robustness. |
| 6 | **Domain-specific Question Answering with Hybrid Search** | Sultania, D., et al. | 2024 | Preprint | [arXiv:2412.03736](https://arxiv.org/abs/2412.03736) | Example of hybrid search using linear score combination and tunable boosts. |
| 7 | **SPLADE: Sparse Lexical and Expansion Model for First Stage Ranking** | Formal, T., et al. | 2021 | SIGIR | [arXiv:2107.05720](https://arxiv.org/abs/2107.05720) | Provides background on learned sparse retrieval and its scoring behavior. |
| 8 | **Dense Passage Retrieval for Open-Domain Question Answering** | Karpukhin, V., et al. | 2020 | EMNLP | [arXiv:2004.04906](https://arxiv.org/abs/2004.04906) | Foundational dense-retrieval work used to establish the semantic retrieval side of the hybrid baseline. |
| 9 | **BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of IR Models** | Thakur, N., et al. | 2021 | NeurIPS | [arXiv:2104.08663](https://arxiv.org/abs/2104.08663) | Benchmark foundation for heterogeneous, zero-shot retrieval evaluation. |
| 10 | **Retrieval-Augmented Generation for Large Language Models: A Survey** | Gao, Y., et al. | 2023 | Preprint | [arXiv:2312.10997](https://arxiv.org/abs/2312.10997) | Places hybrid retrieval, fusion, and reranking within the broader RAG architecture. |
| 11 | **Seven Failure Points When Engineering a Retrieval Augmented Generation System** | Barnett, S., et al. | 2024 | Preprint | [arXiv:2401.05856](https://arxiv.org/abs/2401.05856) | Provides engineering context for retrieval and fusion as important RAG failure points. |
| 12 | **Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection** | Asai, A., et al. | 2024 | ICLR | [arXiv:2310.11511](https://arxiv.org/abs/2310.11511) | Provides context for adaptive retrieval and the downstream importance of retrieval quality. |
| 13 | **Adaptive-k Retrieval for Retrieval-Augmented Generation** | General 2026 Adaptive IR | 2025/2026 | Findings / related adaptive IR | [arXiv:2506.08479](https://arxiv.org/abs/2506.08479) | Related work for dynamic retrieval cutoffs; important for distinguishing dynamic Top-K from dynamic RRF $k$. |
| 14 | **RankZephyr: Effective and Robust Zero-Shot Listwise Reranking is a Breeze!** | Pradeep, R., et al. | 2023 | SIGIR | [arXiv:2312.02724](https://arxiv.org/abs/2312.02724) | Represents a more computationally expensive neural reranking alternative to algorithmic fusion. |

## Classification

### Foundational

Papers 1–3 establish the main RRF and hybrid-Retrieval context.

### Competing or directly relevant methods

Papers 4–6 cover weighted RRF, score/rank fusion comparisons, and practical hybrid score fusion.

### Retrieval baselines and background

Papers 7–12 establish the sparse, dense, benchmark, and RAG context required to position the method.

### Related work

Papers 13–14 cover adaptive retrieval cutoffs and neural reranking, which are relevant alternatives but solve different problems from adaptive RRF smoothing.

## Document Roles

- `Notes/literature_analysis.md` — detailed analysis of what each paper contributes and how it relates to VA-RRF.
- `Papers/literature_review_bibliography.md` — bibliographic index and classification.
- `Decision Log/Research Decision Log.md` — methodological decisions derived from the literature and subsequent research reasoning.
