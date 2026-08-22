# Research Decision Log

This file records methodological decisions and the reasoning behind them. It is the authoritative record of decisions that affect the proposed method, experimental design, or evaluation.

## Retrieval

### Why Hybrid Retrieval?

**Decision:** Use a hybrid retrieval architecture combining dense and sparse retrieval.

**Reason:** Dense retrieval captures semantic and conceptual relationships, while sparse retrieval is stronger at exact keyword and lexical matching. Combining them provides broader retrieval coverage than relying on either modality alone.

**Alternatives considered:** Dense-only retrieval and sparse-only retrieval.

**Why not selected:** Dense-only retrieval can struggle with exact terminology, while sparse-only retrieval can miss synonyms and semantically related terms.

**Limitation:** The two retrievers produce different score distributions, creating a score-compatibility problem when their raw scores are combined.

**Research relevance:** This motivates a fusion method that can combine both retrieval outputs without requiring their raw scores to be directly comparable.

### Why BM25?

**Decision:** Use BM25 as the sparse retrieval baseline.

**Reason:** BM25 is an established and efficient lexical retrieval method that requires no training and provides a strong reproducible baseline across diverse datasets.

**Alternatives considered:** TF-IDF and learned sparse retrieval such as SPLADE.

**Why not selected:** TF-IDF provides a simpler lexical baseline, while SPLADE introduces neural training and greater implementation complexity.

**Limitation:** BM25 depends on lexical overlap and can miss semantically related documents that use different vocabulary.

**Research relevance:** Its lexical behavior complements dense retrieval and makes it appropriate for the hybrid baseline.

### Why Dense Retrieval?

**Decision:** Include a dense retriever alongside BM25.

**Reason:** Dense retrieval represents queries and documents in a semantic vector space, allowing conceptually related documents to be retrieved even when exact terms do not overlap.

**Alternatives considered:** Lexical-only retrieval and late-interaction models such as ColBERT.

**Why not selected:** Lexical-only retrieval cannot capture semantic similarity, while late-interaction models introduce additional memory and computational requirements.

**Limitation:** Dense retrieval can degrade under domain shift, particularly on unfamiliar domains without domain-specific adaptation.

**Research relevance:** This complementary behavior strengthens the case for hybrid retrieval.

### Why RRF?

**Decision:** Use Reciprocal Rank Fusion (RRF) as the baseline fusion method.

**Reason:** Dense and sparse retrievers produce incompatible raw-score distributions. Linear score fusion therefore requires score handling and parameter tuning. RRF combines rankings instead of raw scores, avoiding the need for score-scale alignment.

**Alternatives considered:** Convex Combination (linear score fusion), score-normalization methods such as Min-Max and DBSF, learned fusion, and neural reranking.

**Why not selected:** Linear and normalized score fusion preserve score magnitude but depend on scale compatibility and tuning. Learned fusion and neural reranking introduce training or substantially greater computational overhead.

**Why RRF:** It provides a simple, deterministic, reproducible baseline that can be applied across heterogeneous retrieval settings without training or score normalization.

**Limitation:** RRF is magnitude-blind because it discards raw scores. It also uses a smoothing constant $k$, traditionally set to 60.

**Research relevance:** These limitations motivate investigating whether score-distribution information can be used to adapt the RRF smoothing constant while retaining rank-based robustness.

## Problem

### Why is static $k$ a problem?

### Why not keep $k=60$?

## Existing Solutions

### Offline Grid Search

### Score Normalization (DBSF)

### Neural Rerankers

### Adaptive Top-K

### Learned Fusion Weights

## Proposed Method

### Why Dynamic $k$?

### Why dispersion?

### Why CV?

### Why compare Variance, SD, CV, and IQR?

### Why zero-shot?

### Why unsupervised?

## Evaluation

### Why BEIR?

### Why NDCG?

### Why MRR?

### Why CPU-only?
