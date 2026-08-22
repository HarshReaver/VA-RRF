# Baseline Experiment Analysis

Before testing any dynamic RRF rule, we need to understand how strong the existing methods already are. The purpose of the baseline stage is not to make our method look better. It is to make the comparison fair enough that a reviewer can understand exactly what the dynamic method adds.

## What the baselines should tell us

**Dense-only retrieval** tells us how well the semantic retriever works by itself.

**BM25-only retrieval** tells us how well the lexical retriever works by itself.

**RRF with $k=60$** gives us the standard fixed reference point.

**Other fixed values of $k$** show whether RRF is sensitive to the smoothing constant. A dynamic method should not receive credit simply because it beats the historical default if another fixed value performs better.

**A validation-tuned global $k$** is therefore an important comparison. It represents the best single static setting we can find using validation data. If a dynamic method cannot beat this, its query-level complexity may not be justified.

**Score-normalization fusion** is another useful comparison because it keeps information from the raw scores instead of discarding it. This helps answer whether the better direction is to modify RRF or to normalize and combine the scores directly.

A supervised learned-fusion method or WRRF can be included only if it can be reproduced fairly without adding disproportionate work. It is not essential to answer the main zero-shot research question.

## Fixed $k$ comparison

A small grid such as

$$
k \in \{10,30,60,100,150\}
$$

is a reasonable starting point for seeing how RRF behaves as the decay changes. This is still an experimental choice, not a permanent design decision. We should verify the available runs and validation setup before locking the grid.

The purpose of the grid is to reveal whether the dataset is sensitive to $k$ and whether a dynamic method is doing something beyond selecting a better average value.

## Candidate-method experiment order

Testing every dispersion statistic against every possible mapping function at once would create a large matrix and make the results difficult to interpret. A staged experiment is cleaner:

```text
Fixed RRF baseline
       ↓
Compare candidate dispersion signals
       ↓
Keep useful candidates
       ↓
Compare simple mappings from signal to k
       ↓
Lock the candidate method
       ↓
Evaluate on additional datasets
       ↓
Run ablations and robustness checks
```

The dense retriever, sparse retriever, retrieval depth, run files, and evaluation metrics should stay fixed while a specific comparison is being made.

## Controls

The same pre-computed dense and sparse runs should be reused across all fusion methods. The candidate-pool depth used for calculating score statistics should also remain fixed within an experiment, because changing the pool changes the measured score distribution.

Any mapping boundaries or other tunable parameters should be selected using validation data and then frozen before the final test evaluation. This prevents test data from influencing the method.

## What counts as success?

Beating $k=60$ alone is not enough. A convincing result would show improvement over both the standard $k=60$ baseline and the best validation-tuned global $k$ while keeping the added fusion computation small.

Matching the tuned global $k$ would be interesting but weaker: it could mean that the dynamic rule has mainly found a better average setting rather than providing useful query-level adaptation.

Performing worse than $k=60$ would be evidence against the dynamic rule. Performing worse than both dense-only and BM25-only retrieval would be stronger evidence that the chosen adaptation signal is actively damaging the fusion.

## Important scope rule

This remains an isolated retrieval-fusion experiment. We do not need to build a complete RAG application or add LLM generation to answer the central research question. The final experimental choices for datasets, fixed-$k$ grid, dispersion statistics, and mapping functions remain open until the compatible pre-computed runs have been checked.
