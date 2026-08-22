# Research Decision Log

## Retrieval

### Why Hybrid Retrieval?

### Why BM25?

### Why Dense Retrieval?

## Decision 1 — Why RRF?

**Decision:** Use Reciprocal Rank Fusion (RRF) as the baseline for hybrid retrieval.

**Reason:** Dense and sparse retrievers produce incompatible score distributions. Linear score fusion therefore requires score normalization and weight tuning. RRF avoids this by combining rankings rather than raw scores, making it simple, zero-shot, and robust across different retrievers and datasets.

**Alternatives considered:** Convex Combination / linear score fusion, score-normalization methods such as Min-Max and DBSF, and learned fusion/reranking.

**Why not selected:** Linear and normalized score fusion retain useful score-magnitude information but depend on scale compatibility and tuning. Learned approaches introduce training and additional computational overhead.

**Why RRF:** RRF provides a strong baseline without score normalization or training while remaining applicable across heterogeneous retrieval settings.

**Limitation:** RRF is magnitude-blind because it discards raw scores. A large score gap and a small score gap can produce the same rank relationship. RRF also uses a fixed smoothing constant \(k\), traditionally set to 60.

**Research relevance:** This limitation motivates investigating whether score-distribution information can be used to adapt the RRF smoothing constant while retaining the robustness of rank-based fusion.

**Status:** Confirmed.

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
