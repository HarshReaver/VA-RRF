# Research Decision Log

This file records methodological decisions and the reasoning behind them. It is the authoritative record of decisions that affect the proposed method, experimental design, or evaluation.

## Retrieval

### Why Hybrid Retrieval?

**Decision:** Use a hybrid retrieval architecture combining dense and sparse retrieval.

**Reason:** Dense retrieval captures semantic relationships, while sparse retrieval is stronger at exact lexical matching. Combining them provides broader retrieval coverage than relying on either modality alone.

**Alternatives considered:** Dense-only retrieval and sparse-only retrieval.

**Why not selected:** Dense-only retrieval can miss exact terminology, while sparse-only retrieval can miss synonyms and semantically related documents.

**Limitation:** The two retrievers produce different score distributions, creating a score-compatibility problem when raw scores are combined.

**Research relevance:** This motivates a fusion method that can combine both retrieval outputs without requiring their raw scores to be directly comparable.

### Why BM25?

**Decision:** Use BM25 as the sparse retrieval baseline.

**Reason:** BM25 is an established, efficient lexical retrieval method that requires no training and provides a strong reproducible baseline.

**Alternatives considered:** TF-IDF and learned sparse retrieval such as SPLADE.

**Why not selected:** TF-IDF is a simpler baseline, while SPLADE introduces neural training and greater implementation complexity.

**Limitation:** BM25 depends on lexical overlap and can miss semantically related documents that use different vocabulary.

**Research relevance:** Its lexical behavior complements dense retrieval and makes it appropriate for the hybrid baseline.

### Why Dense Retrieval?

**Decision:** Include a dense retriever alongside BM25.

**Reason:** Dense retrieval represents queries and documents in a semantic vector space, allowing conceptually related documents to be retrieved even when exact terms do not overlap.

**Alternatives considered:** Lexical-only retrieval and late-interaction models such as ColBERT.

**Why not selected:** Lexical-only retrieval cannot capture semantic similarity, while late-interaction models add memory and computational requirements beyond the intended baseline.

**Limitation:** Dense retrieval can degrade under domain shift, particularly on unfamiliar domains without adaptation.

**Research relevance:** Its complementary behavior strengthens the case for hybrid retrieval.

### Why RRF?

**Decision:** Use Reciprocal Rank Fusion (RRF) as the baseline fusion method.

**Reason:** Dense and sparse retrievers produce incompatible raw-score distributions. Linear score fusion therefore requires score handling and parameter tuning. RRF combines rankings instead of raw scores, avoiding the need for score-scale alignment.

**Alternatives considered:** Convex Combination, score-normalization methods such as Min-Max and DBSF, learned fusion, and neural reranking.

**Why not selected:** Score-based approaches preserve magnitude but depend on scale compatibility and tuning. Learned and neural approaches introduce additional training or computational overhead.

**Why RRF:** It provides a simple, deterministic, reproducible baseline that can be applied across heterogeneous retrieval settings without training or raw-score normalization.

**Limitation:** RRF is magnitude-blind because it discards raw scores. It also uses a smoothing constant $k$, traditionally set to 60.

**Research relevance:** These limitations motivate investigating whether score-distribution information can be used to adapt the RRF smoothing constant while retaining rank-based robustness.

## Problem

### Why is static $k$ a problem?

**Decision:** Treat the fixed RRF smoothing constant as a research problem rather than assuming $k=60$ is universally suitable.

**Reason:** The constant $k$ controls how quickly RRF scores decay with rank. Smaller values emphasize top ranks more strongly; larger values flatten the rank contribution. The original RRF value of $k=60$ was empirically selected, not derived from a theoretical optimum.

**Evidence:** Bruch et al. (2023) report sensitivity to RRF parameters, including $k$, and show that properly tuned alternatives can outperform untuned RRF in some settings.

**Limitation of a fixed value:** Different retrieval systems and query distributions can have different ranking behavior, so one global decay setting may not be optimal everywhere.

**Research relevance:** This creates a narrow question: can the rank-decay behavior be adapted to the current retrieval situation without replacing RRF itself?

### Why not keep $k=60$?

**Decision:** Keep $k=60$ as the primary baseline, but do not assume it is universally optimal.

**Reason:** $k=60$ is the established RRF default and is useful as a consistent reference point. It remains reasonable when no tuning data is available or when simplicity is the main objective.

**Why it may be insufficient:** Its value originated from empirical experimentation and is fixed for every query. Literature shows that tuning RRF parameters can change performance, so a universal fixed value is not guaranteed to be best.

**Research relevance:** VA-RRF must therefore be compared directly against standard RRF with $k=60$, rather than treating standard RRF as an invalid baseline.

## Existing Solutions

### Offline Grid Search

**Approach:** Test multiple candidate values of $k$ on a validation set and select the value that gives the best retrieval metric.

**Why it is used:** It is simple and can optimize RRF for a known target distribution.

**Advantage:** No new model architecture is required; only the RRF parameter is tuned.

**Limitation:** The selected value is typically a global setting for the validation distribution. It requires relevance judgments for metric-based tuning and does not naturally adapt to individual queries at inference time.

**Why it does not fully solve the problem:** A value optimized offline for the average query can remain suboptimal for a particular query with a different retrieval-score or rank distribution.

### Score Normalization (DBSF)

**Approach:** Normalize or standardize scores from each retriever before combining them so that their magnitudes become comparable.

**Why it is used:** It preserves score-magnitude information that RRF discards while addressing differences in score scale.

**Advantage:** Confidence gaps between documents remain available to the fusion method.

**Limitation:** Score-based fusion can be sensitive to the statistical shape of the score distribution and to outliers. Normalization changes the raw score space rather than the rank-decay behavior of RRF.

**Why it does not fully solve the problem:** It solves score compatibility by moving away from pure rank fusion; it does not make the RRF smoothing constant adaptive.

### Neural Rerankers

**Approach:** Apply a neural or LLM-based model to a retrieved candidate set and rerank the documents using deeper query-document understanding.

**Why it is used:** Neural rerankers can capture semantic relationships that simple rank-based fusion cannot.

**Advantage:** They can substantially improve ranking quality when sufficient compute and an appropriate model are available.

**Limitation:** They add model inference cost, latency, and implementation complexity. Some learned approaches also require training or task-specific data.

**Why it does not fully solve the problem:** Neural reranking addresses a broader semantic-ranking problem rather than specifically improving the RRF rank-decay mechanism. It is therefore a different design trade-off, not a direct replacement for the proposed lightweight fusion adjustment.

### Adaptive Top-K

**Approach:** Adapt the number of retrieved documents or chunks retained for downstream processing according to the query or retrieval conditions.

**Why it is used:** Fixed Top-K can pass unnecessary context for simple queries or insufficient context for complex queries. Adaptive Top-K addresses this context-quantity problem by changing the cutoff.

**Advantage:** It can reduce downstream token usage and latency while retaining more context when needed.

**Limitation:** It does not change the underlying ranking. If an important document is ranked poorly before the cutoff is applied, Adaptive Top-K does not correct that ranking.

**Why it is different from dynamic RRF $k$:** RRF $k$ controls the decay of rank contributions during list fusion. Adaptive Top-K controls how many documents are retained after ranking. They act at different stages of the retrieval pipeline.

### Learned Fusion Weights

**Approach:** Learn or tune weights that control the contribution of different retrieval channels.

**Why it is used:** Dense and sparse retrievers can have different strengths, so weighting their contributions can improve the final ranking.

**Advantage:** Properly tuned or learned fusion can exploit differences in retriever quality and may outperform an untuned RRF baseline when suitable validation or training data is available.

**Limitation:** Learned approaches can require relevance judgments, training or validation data, tuning, and potentially retraining when the retrieval environment changes.

**Why it does not fully solve the problem:** It represents a data-driven change in channel importance rather than a query-specific adaptation of RRF's rank-decay function without supervised training.

## Our Solution

### Why Dynamic $k$?

### Why dispersion?

### Why CV?

### Why compare Variance, SD, CV, and IQR?

### Why zero-shot and unsupervised?

## Evaluation

### Why BEIR?

### Why NDCG?

### Why MRR?

### Why CPU-only?
