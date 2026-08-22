# VA-RRF Literature Analysis

This document contains the detailed paper-by-paper analysis used to position Variance-Aware Reciprocal Rank Fusion (VA-RRF). It preserves the literature-derived reasoning and separates it from the shorter bibliographic index in `Papers/literature_review_bibliography.md`.

## 1. Reciprocal Rank Fusion outperforms Condorcet and individual Rank Learning Methods (Cormack et al., 2009)

* **Fusion Method Used:** **Reciprocal Rank Fusion (RRF)**, introducing the classic formula:

  $$
  \mathrm{RRF}(d)=\sum_{m \in M}\frac{1}{k+r_m(d)}
  $$

  where $k$ is the smoothing constant.
* **Why RRF was Selected:** To find a "simple method for combining the document rankings from multiple IR systems" that did not require training or complex configuration.
* **Problem RRF Solved:** It bypassed the high computational cost and heavy training data requirements of learning-to-rank algorithms on datasets like LETOR 3, while consistently yielding "better results than any individual system, and better results than the standard method Condorcet Fuse".
* **Limitation of Previous Methods Addressed:** Standard rank aggregation methods like Condorcet Fuse were computationally complex and highly sensitive to poorly performing individual retrieval runs.
* **Remaining Limitation of RRF:** The constant $k$ was arbitrarily set to 60 as a heuristic baseline. It remained static and empirical, with no mechanism to adapt to different retrieval runs or document score distributions.

---

## 2. An Analysis of Fusion Functions for Hybrid Retrieval (Bruch et al., 2023)

* **Fusion Method Used:** Compared **Convex Combination (CC)** (linear combination of normalized scores) against **Reciprocal Rank Fusion (RRF)**.
* **Why RRF was Selected:** Evaluated as a strong baseline for hybrid search due to its parameter simplicity and resistance to score-scale problems.
* **Problem RRF Solved:** Under CC, lexical and semantic scores are scale-incompatible, which typically requires score normalization. RRF bypasses score-scale disparities by relying on ranks.
* **Limitation of Previous Methods Addressed:** Linear score fusion traditionally required choosing a score-normalization approach. The study also examines how CC behaves under different normalization choices.
* **Remaining Limitation of RRF:** The study reports that RRF is sensitive to its parameters, including $k$, and shows that a properly tuned linear combination can outperform RRF in some in-domain and out-of-domain settings.

---

## 3. RAG-Fusion: A New Take on Retrieval-Augmented Generation (Rackauckas, 2024)

* **Fusion Method Used:** **Reciprocal Rank Fusion (RRF)** combined with query generation.
* **Why RRF was Selected:** Used to fuse retrieved document lists produced from multiple generated queries representing different perspectives.
* **Problem RRF Solved:** Traditional RAG is limited by a single user query. RAG-Fusion generates multiple queries and uses RRF to combine their retrieval results without requiring raw-score alignment.
* **Limitation of Previous Methods Addressed:** It addresses the narrow context and vocabulary mismatch of single-query retrieval.
* **Remaining Limitation of RRF:** Because RRF is magnitude-blind, it does not directly represent the relevance strength of each generated query. Poorly relevant generated queries can therefore contribute documents to the fused ranking.

---

## 4. Weighted Reciprocal Rank Fusion RAG for Context-Aware DoS Attack Mitigation (Kafi & Saha, 2026)

* **Fusion Method Used:** **Weighted Reciprocal Rank Fusion (WRRF)**.
* **Why RRF was Selected:** Rank-based fusion provides a scale-independent way to merge retrieval sources, while standard RRF does not distinguish the reliability of different sources.
* **Problem RRF Solved:** Standard RRF treats retrieval sources equally. WRRF addresses this by adding confidence weights to the RRF contribution of different sources.
* **Limitation of Previous Methods Addressed:** It provides a mechanism for favoring higher-confidence retrieval systems over noisier sources.
* **Remaining Limitation of RRF:** Weighting the lists does not by itself make the rank-smoothing constant adaptive, so the underlying static rank-smoothing assumption remains.

---

## 5. Comparative Evaluation of Rank and Score Fusion Methods for Hybrid Search (Cormack & Clarke et al., 2025)

* **Fusion Method Used:** Compared **Distribution-Based Score Fusion (DBSF)** with **Reciprocal Rank Fusion (RRF)**.
* **Why RRF was Selected:** Used as the rank-based representative because rank fusion is less exposed to raw-score outliers.
* **Problem RRF Solved:** It avoids the outlier sensitivity that can affect score-based fusion when an anomalous score dominates the combined ranking.
* **Limitation of Previous Methods Addressed:** Score-based fusion can be sensitive to score scaling and outliers.
* **Remaining Limitation of RRF:** RRF is **magnitude blind**. It discards the relative confidence encoded in raw scores. A large score gap and a small score gap can produce the same rank relationship.

---

## 6. Domain-specific Question Answering with Hybrid Search (Sultania et al., 2024)

* **Fusion Method Used:** **Linear combination of scores** with tunable boost parameters for BM25 scores, dense retrieval similarity, and URL host matching.
* **Why RRF was not selected:** The approach required direct control over heterogeneous signals, including enterprise-specific rule boosts, which linear scoring can incorporate explicitly.
* **Problem Linear Fusion Solved:** It combined lexical, semantic, and metadata signals for domain-specific question answering.
* **Limitation of Previous Methods Addressed:** It moved beyond single-retriever search by combining complementary retrieval signals.
* **Remaining Limitation:** Manual weight and boost tuning can become difficult to maintain when score distributions or domains change. This provides a practical contrast with the scale-free RRF baseline.

---

## 7. SPLADE: Sparse Lexical and Expansion Model for First Stage Ranking (Formal et al., 2021)

* **Fusion Method Used:** Learned sparse representations with explicit sparsity regularization and a log-saturation effect on term weights.
* **Why RRF is relevant:** Learned-sparse models produce scoring behavior that is not naturally comparable to dense cosine or inner-product scores.
* **Problem RRF addresses in this context:** Rank fusion can combine learned-sparse and dense retrieval results without requiring their raw score scales to be aligned.
* **Limitation of Previous Methods Addressed:** It avoids direct score dominance when combining heterogeneous retrieval scores.
* **Remaining Limitation of RRF:** Rank fusion discards the detailed term-weight information encoded by the sparse model.

---

## 8. Dense Passage Retrieval for Open-Domain Question Answering (Karpukhin et al., 2020)

* **Fusion Method Used:** Dense vector retrieval using a dual-encoder framework.
* **Why RRF is relevant:** Dense retrieval supplies the semantic component of a hybrid system and can be combined with lexical retrieval through rank-based fusion.
* **Problem Dense Retrieval Solved:** It addresses vocabulary mismatch by retrieving passages through learned semantic representations rather than exact lexical overlap.
* **Limitation of Previous Methods Addressed:** It provides semantic retrieval capabilities beyond traditional sparse matching.
* **Remaining Limitation for Fusion:** Dense retrieval scores cannot automatically be assumed to be on the same scale as BM25 or other sparse scores, motivating scale-independent fusion methods.

---

## 9. BEIR: A Heterogeneous Benchmark for Zero-shot Evaluation of IR Models (Thakur et al., 2021)

* **Fusion Method Used:** Evaluates diverse lexical, sparse, dense, late-interaction, and reranking architectures across heterogeneous datasets.
* **Why RRF is relevant:** A zero-shot benchmark makes a scale-independent hybrid baseline useful because domain-specific score normalization or tuning cannot be assumed.
* **Problem BEIR Solved:** It addressed the limitations of evaluating retrieval models only on homogeneous or in-domain datasets.
* **Limitation of Previous Methods Addressed:** It provides a broader test of generalization across retrieval tasks and domains.
* **Remaining Limitation for RRF:** A fixed smoothing constant may not be equally suitable for every dataset or retrieval distribution, making BEIR relevant for testing robustness to heterogeneous conditions.

---

## 10. Retrieval-Augmented Generation for Large Language Models: A Survey (Gao et al., 2023)

* **Fusion Method Used:** Places retrieval, reranking, and fusion within broader RAG architectures.
* **Why RRF is relevant:** Hybrid retrieval and fusion can be used to combine complementary lexical and semantic contexts before generation.
* **Problem RAG Fusion Addresses:** Multi-source retrieval can provide broader context than relying on a single retrieval mechanism.
* **Limitation of Previous Methods Addressed:** Hybrid retrieval can mitigate the weaknesses of relying on only lexical or only semantic retrieval.
* **Remaining Limitation of RRF:** Static fusion does not explicitly adapt its behavior to the characteristics of each query.

---

## 11. Seven Failure Points When Engineering a Retrieval Augmented Generation System (Barnett et al., 2024)

* **Fusion Method Used:** Discusses retrieval, mathematical fusion, and reranking as engineering components of RAG systems.
* **Why RRF is relevant:** First-stage retrieval quality is a major determinant of downstream RAG quality.
* **Problem Addressed:** The work identifies practical failure points that can arise when building production RAG systems.
* **Limitation of Previous Methods Addressed:** It emphasizes the need to treat retrieval quality as a core system concern rather than an isolated component.
* **Remaining Limitation for RRF:** A fixed fusion strategy can retain rigid assumptions about how retrieval results should be combined.

---

## 12. Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection (Asai et al., 2024)

* **Fusion Method Used:** An LLM learns to retrieve passages and critique them using reflection tokens.
* **Why RRF is relevant:** Better first-stage retrieval can provide higher-quality contexts before generation and reduce the burden placed on downstream retrieval or critique mechanisms.
* **Problem Self-RAG Addresses:** Standard RAG retrieves a fixed set of passages without dynamically evaluating whether additional retrieval is necessary.
* **Limitation of Previous Methods Addressed:** Self-RAG introduces adaptive retrieval and self-reflection instead of treating every retrieval result identically.
* **Remaining Limitation for RRF:** Standard RRF does not itself provide a learned confidence signal indicating whether the fused retrieval result is sufficient.

---

## 13. Adaptive-k Retrieval for Retrieval-Augmented Generation (General 2026 Adaptive IR)

* **Fusion Method Used:** Dynamically varies the **Top-K retrieval cutoff** based on query characteristics.
* **Why RRF is relevant:** It clarifies an important terminology distinction: dynamic **Top-K** controls how many retrieved documents are retained, whereas VA-RRF's dynamic **$k$** concerns the RRF smoothing constant.
* **Problem Addressed:** Fixed-K retrieval can pass unnecessary context to the LLM or omit the amount of context needed for a particular query.
* **Limitation of Previous Methods Addressed:** Adaptive retrieval cutoffs respond to query complexity rather than using one fixed cutoff.
* **Remaining Limitation for RRF:** Changing Top-K does not change the underlying fusion function; poor fusion rankings can still place relevant documents too low.

---

## 14. RankZephyr: Effective and Robust Zero-Shot Listwise Reranking is a Breeze! (Pradeep et al., 2023)

* **Fusion Method Used:** **Zero-shot listwise reranking** using an instruction-tuned open-source LLM.
* **Why RRF is relevant:** Algorithmic fusion can act as an inexpensive first-stage method for reducing the candidate set before expensive neural reranking.
* **Problem RankZephyr Addresses:** It improves ranking quality through listwise LLM reranking without requiring supervised task-specific training.
* **Limitation of Previous Methods Addressed:** It provides a zero-shot neural alternative to conventional reranking approaches.
* **Remaining Limitation of RRF:** RRF is computationally inexpensive but does not inspect semantic relationships between candidate documents as deeply as an LLM reranker.

---

# Synthesis: From Score Fusion to Rank Fusion

The literature reveals a recurring trade-off between **score information** and **score compatibility**.

Linear score fusion preserves the magnitude of each retriever's relevance signal, but heterogeneous retrievers can produce incompatible score distributions. This creates a need for normalization and weight selection.

RRF avoids that compatibility problem by discarding raw scores and combining ranks. Its main strength is therefore also its main limitation: it gains robustness by giving up score magnitude information.

The literature also identifies two related directions that do not directly solve the same problem as VA-RRF. **Weighted RRF** changes the contribution of retrieval systems, while **adaptive Top-K** changes how many retrieved documents are retained. Neither directly changes the RRF smoothing constant based on the current retrieval distribution.

This leaves the research question investigated by VA-RRF: whether information from the current retrieval-score distribution can be used to adapt the RRF smoothing constant while preserving the scale-independent rank-based structure of RRF.
