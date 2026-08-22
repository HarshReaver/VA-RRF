# Evaluation Analysis

## Why BEIR?

BEIR is a benchmark for evaluating information-retrieval systems across different datasets, tasks, and domains. This diversity is useful because our research should not appear to work only on one narrow collection.

A full run across every BEIR dataset is not necessary for the first experiment. It would add a large amount of downloading, indexing, and runtime work. A smaller, deliberately chosen subset is more practical, as long as the datasets represent meaningfully different retrieval conditions.

The final subset should be chosen after checking which compatible pre-computed dense and sparse runs are available and how long the evaluation takes.

## Why NDCG?

NDCG measures both relevance and rank position. A relevant document near the top contributes more than the same document near the bottom.

For example, suppose two systems both retrieve the same three useful papers in their top 10. One puts them at ranks 1, 2, and 3; the other puts them at ranks 8, 9, and 10. NDCG distinguishes these rankings even though the retrieved set is the same.

NDCG@10 restricts the calculation to the first 10 results. This gives us a consistent view of the part of the ranking where retrieval quality matters most.

A limitation is that NDCG is most informative when relevance labels contain grades. With only binary relevance labels, it still measures ranking quality, but its graded-relevance advantage is reduced.

## Why MRR?

MRR focuses on the first relevant result. For one query, if the first relevant document is at rank $r$, the reciprocal rank is:

$$
RR = \frac{1}{r}
$$

MRR averages that value over all queries.

For example, a first relevant result at rank 1 gives $1$, while rank 5 gives $0.2$. MRR therefore answers a simple question: how quickly does the system find at least one useful document?

MRR and NDCG@10 measure different aspects of retrieval. MRR mainly cares about the first useful result, while NDCG@10 evaluates the quality and order of the broader top-10 set. Using both helps us detect cases where a method improves the first hit but harms the rest of the list, or improves the overall list without moving the first hit much.

MRR is not a good standalone measure when a task requires several relevant documents, because additional relevant documents after the first one do not increase the reciprocal rank.

## Why CPU-only?

The central experiment concerns the fusion stage. If dense and sparse retrieval outputs are already available, the fusion code mainly performs ranking, arithmetic, and simple statistics on numerical arrays. These operations can be run on a CPU.

This does not mean the entire retrieval pipeline is CPU-only. Creating embeddings and generating dense retrieval outputs can require substantial model computation. We can isolate the fusion study by using pre-computed retrieval runs when they are available.

We should measure the actual CPU time of the fusion and candidate dynamic-$k$ calculations instead of assuming the overhead is negligible.

## Evaluation design

The experiment should remain an **isolated retrieval-fusion study**. A complete RAG application and LLM answer generation are not necessary for answering the central research question.

```text
Pre-computed dense run + pre-computed sparse run
                    ↓
              Fusion methods
                    ↓
              Ranked documents
                    ↓
             BEIR relevance data
                    ↓
             NDCG@10 + MRR
```

We should start with a small, diverse subset of BEIR. The final subset should be selected using three practical criteria:

- meaningful differences in domain or task
- compatible pre-computed retrieval runs
- reasonable storage and runtime requirements

## What should be compared?

The first stage should establish the behaviour of standard RRF before any dynamic method is introduced.

A sensible sequence is:

1. Standard RRF with the established $k=60$ baseline.
2. Other fixed $k$ values to understand how sensitive the baseline is.
3. Relevant existing fusion baselines that can be reproduced from the available runs.
4. Candidate dynamic signals such as CV or normalized SD.
5. Candidate mappings only after the signal itself appears useful.

This order matters. We should not tune a complex mapping before knowing whether the underlying signal contains useful information.

## What we do not need yet

We do not need to build a complete RAG application, run an LLM generator, evaluate generated answers with ROUGE or BLEU, or deploy a production vector database. Those additions would increase engineering work without directly answering the first research question: whether a different fusion rule improves retrieval ranking.

## Current status

The evaluation framework is decided at a high level, but the exact dataset subset, fixed-$k$ comparison range, existing baselines, candidate dispersion metrics, and mapping functions remain experimental choices. They should be finalized only after we inspect the available pre-computed runs and verify what can be reproduced reliably.
