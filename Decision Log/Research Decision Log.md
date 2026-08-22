# Research Decision Log

## Retrieval

### Why Hybrid Retrieval?

**Decision:** Use a hybrid retrieval architecture combining dense and sparse retrieval.

**Reason:** Dense retrieval captures semantic and conceptual relationships, while sparse retrieval is stronger at exact keyword and lexical matching. Combining them reduces the weaknesses of relying on either modality alone. :contentReference[oaicite:1]{index=1}

**Alternatives considered:** Dense-only retrieval and sparse-only retrieval.

**Why not selected:** Dense-only retrieval can struggle with exact terminology and domain shifts, while sparse-only retrieval cannot reliably capture synonyms or semantic relationships. :contentReference[oaicite:2]{index=2}

**Why Hybrid Retrieval:** The complementary strengths of both modalities provide broader retrieval coverage.

**Limitation:** Combining the two creates a score-compatibility problem because their raw score distributions differ substantially.

**Research relevance:** This creates the need for a robust fusion method that can combine both retrieval outputs without distorting their rankings.

**Status:** Confirmed.


### Why BM25?

**Decision:** Use BM25 as the sparse retrieval baseline.

**Reason:** BM25 is an established, efficient lexical retrieval method with term-frequency saturation and document-length normalization. It requires no training and provides a strong reproducible baseline across diverse datasets. :contentReference[oaicite:3]{index=3}

**Alternatives considered:** TF-IDF and learned sparse retrieval such as SPLADE.

**Why not selected:** TF-IDF provides simpler lexical matching, while SPLADE introduces neural training and greater implementation complexity.

**Why BM25:** It gives us a simple, low-cost sparse baseline while remaining strong enough for heterogeneous retrieval experiments.

**Limitation:** BM25 relies on lexical overlap and can miss synonyms and semantically related terms that use different vocabulary. :contentReference[oaicite:4]{index=4}

**Research relevance:** Its complementary behavior with dense retrieval makes BM25 appropriate for the hybrid setting.

**Status:** Confirmed.


### Why Dense Retrieval?

**Decision:** Include a dense retriever alongside BM25.

**Reason:** Dense retrieval represents queries and documents in a semantic vector space, allowing it to retrieve conceptually related documents even when exact terms do not overlap. :contentReference[oaicite:5]{index=5}

**Alternatives considered:** Lexical-only retrieval and late-interaction models such as ColBERT.

**Why not selected:** Lexical-only retrieval cannot capture semantic similarity, while late-interaction models introduce additional memory and computational requirements. :contentReference[oaicite:6]{index=6}

**Why Dense Retrieval:** It complements BM25 by recovering semantic matches that lexical retrieval may miss.

**Limitation:** Dense retrieval can degrade under domain shift, particularly when evaluated on unfamiliar domains without domain-specific adaptation. :contentReference[oaicite:7]{index=7}

**Research relevance:** This reinforces the need for hybrid retrieval rather than relying on dense retrieval alone.

**Status:** Confirmed.


## Why RRF Instead of Learned Fusion?

**Decision:** Use Reciprocal Rank Fusion (RRF) as the primary fusion method.

**Reason:** RRF combines ranked results without requiring raw-score alignment, training, or learned fusion parameters. Its rank-based formulation makes it robust to score-scale differences and outliers. :contentReference[oaicite:8]{index=8}

**Alternatives considered:** Learned/weighted score fusion and neural reranking.

**Why not selected:** Score-based approaches require score handling and often parameter tuning, while neural rerankers introduce substantially greater computational and implementation overhead.

**Why RRF:** It provides a simple, deterministic and reproducible baseline suitable for zero-shot evaluation.

**Limitation:** RRF discards raw score magnitudes and is therefore magnitude-blind. It also depends on a smoothing constant \(k\), traditionally set to 60. :contentReference[oaicite:9]{index=9}

**Research relevance:** These limitations provide the motivation for investigating whether score-distribution information can improve RRF without abandoning its rank-based robustness.

**Status:** Confirmed.


## Why Zero-Shot / Unsupervised?

**Decision:** Evaluate the proposed method without training or domain-specific labelled data.

**Reason:** A zero-shot setup allows the same method to be evaluated across heterogeneous datasets without learning dataset-specific fusion parameters.

**Alternatives considered:** Supervised learning-to-rank and dataset-specific parameter tuning.

**Why not selected:** These approaches can improve in-domain performance but introduce training data, tuning, and greater risk of dataset-specific optimization.

**Why Zero-Shot:** It directly tests whether the proposed fusion strategy can adapt from the retrieval results themselves rather than relying on prior training.

**Limitation:** Avoiding training also prevents adaptation to domain-specific patterns and may reduce peak in-domain performance. :contentReference[oaicite:10]{index=10}

**Research relevance:** This makes self-adaptation from the current query's retrieval characteristics central to our research.

**Status:** Confirmed.


##  Why RRF?

**Decision:** Use Reciprocal Rank Fusion (RRF) as the baseline for hybrid retrieval.

**Reason:** Dense and sparse retrievers produce incompatible score distributions. Linear score fusion therefore requires score normalization and weight tuning. RRF avoids this by combining rankings rather than raw scores, making it simple, zero-shot, and robust across different retrievers and datasets.

**Alternatives considered:** Convex Combination / linear score fusion, score-normalization methods such as Min-Max and DBSF, and learned fusion/reranking.

**Why not selected:** Linear and normalized score fusion retain useful score-magnitude information but depend on scale compatibility and tuning. Learned approaches introduce training and additional computational overhead.

**Why RRF:** RRF provides a strong baseline without score normalization or training while remaining applicable across heterogeneous retrieval settings.

**Limitation:** RRF is magnitude-blind because it discards raw scores. A large score gap and a small score gap can produce the same rank relationship. RRF also uses a fixed smoothing constant \(k\), traditionally set to 60.

**Research relevance:** This limitation motivates investigating whether score-distribution information can be used to adapt the RRF smoothing constant while retaining the robustness of rank-based fusion.

## Problem

### Why is static k a problem?

### Why not keep k=60?

## Existing Solutions

### Offline Grid Search

### Score Normalization (DBSF)

### Neural Rerankers

### Adaptive Top-K

### Learned Fusion Weights

## Our Solution

### Why Dynamic k?

### Why dispersion?

### Why CV?

### Why compare Variance/SD/CV/IQR?

### Why zero-shot?

### Why unsupervised?

## Evaluation

### Why BEIR?

### Why NDCG?

### Why MRR?

### Why CPU-only?

