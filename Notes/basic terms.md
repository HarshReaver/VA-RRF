# Retrieval Fusion: Basic Terms

## 1. Linear Score Fusion

Linear score fusion combines the **raw scores** produced by different retrievers using weights.

$$
S(d) = \alpha S_1(d) + (1-\alpha)S_2(d)
$$

The main advantage is that it preserves score magnitude: a large confidence gap can influence the final score more strongly than a small gap.

The main difficulty is that different retrievers can produce scores on incompatible scales. Their scores may therefore need normalization, and the fusion weight may need tuning.

## 2. Reciprocal Rank Fusion (RRF)

RRF ignores raw score magnitude and combines the **rank positions** assigned by different retrievers.

$$
\mathrm{RRF}(d) = \sum_{m \in M} \frac{1}{k + \mathrm{rank}_m(d)}
$$

where $M$ is the set of retrieval systems, $\mathrm{rank}_m(d)$ is the rank of document $d$ in system $m$, and $k$ is the smoothing constant.

The standard formulation traditionally uses $k = 60$.

### Why RRF is useful

Because it uses ranks rather than raw scores, RRF does not require the score scales of different retrievers to be aligned. This makes it particularly convenient for hybrid retrieval involving sparse and dense systems.

### Main limitation

RRF is **magnitude-blind**. It cannot distinguish between a large and a small score gap when the resulting rank order is identical.

For example, a retriever ranking two documents with scores $0.99$ and $0.50$ produces the same rank relationship as scores $0.99$ and $0.98$. RRF sees only ranks, not the size of the gap.

## 3. The RRF Smoothing Constant $k$

The smoothing constant $k$ controls how strongly rank position affects the RRF contribution.

$$
\mathrm{Contribution}(r) = \frac{1}{k+r}
$$

A **small $k$** makes the decay steeper. Rank 1 receives a noticeably larger contribution than lower ranks, so the top of each retrieval list matters more.

A **large $k$** makes the decay flatter. The difference between nearby ranks becomes smaller, allowing documents that appear consistently across several lists to compete more effectively.

### Simple example

Imagine two documents:

- Document A appears at rank 1 in one retriever and nowhere else.
- Document B appears around rank 5 in several retrievers.

With a small $k$, A receives a strong advantage from its rank-1 position. With a larger $k$, B's repeated mid-level appearances become relatively more competitive.

The exact effect depends on all retrieval lists, so $k$ should be understood as controlling the **shape of rank decay**, not as a direct measure of document relevance.

## 4. Static vs Dynamic $k$

A **static $k$** uses the same value for every query.

A **dynamic $k$** changes according to information available for the current query or retrieval result. The idea is similar to using different camera exposure settings for different lighting conditions: one fixed setting is simple, but different conditions may require different settings.

### Offline global tuning

A common alternative to the default $k=60$ is to test many candidate values on a validation dataset and keep the value that performs best on average. This produces a better **global setting**, but the chosen value still remains fixed at inference time.

### Query-dependent $k$

A query-dependent method calculates or selects $k$ separately for each query. It can therefore react to differences between retrieval runs instead of forcing every query to use the same rank-decay behavior.

### Per-modality $k$

A modality-specific method can assign different values such as $k_{dense}$ and $k_{sparse}$. This matters when the dense and sparse retrieval lists show different behavior for the same query.

## 5. Rank Decay

**Rank decay** describes how quickly a retrieval method reduces the contribution of lower-ranked documents.

For RRF, the decay is determined by:

$$
\frac{1}{k+r}
$$

A steep decay strongly favors the first few ranks. A flat decay gives lower-ranked results relatively more influence.

In real life, this is like deciding how much attention to give a recommendation based on its position in a ranked list. A steep rule says "first place matters a lot more"; a flatter rule says "several good-looking options should remain competitive."

## 6. Score Distribution

A **score distribution** describes how the retrieval scores are spread across the returned documents.

For example:

- Scores such as $0.99, 0.98, 0.97, 0.96$ form a relatively tight group.
- Scores such as $0.99, 0.70, 0.40, 0.15$ show a much sharper drop.

The first pattern suggests that the retrieved candidates have similar scores. The second suggests that the top result is separated much more strongly from the rest.

Our research asks whether this information can help determine how strongly RRF should emphasize rank positions.

## 7. Variance and Standard Deviation

**Variance** measures how far scores spread from their average, using squared differences.

**Standard deviation (SD)** is the square root of variance and expresses the spread in the same units as the scores.

A larger SD means the scores are more dispersed; a smaller SD means they are more tightly clustered.

These are descriptive statistics, not measures of retrieval relevance by themselves. In this research they are candidate signals that may describe the local shape of a retrieval score list.

## 8. Coefficient of Variation (CV)

The **Coefficient of Variation (CV)** measures spread relative to the mean:

$$
CV = \frac{\sigma}{\mu}
$$

where $\sigma$ is the standard deviation and $\mu$ is the mean.

This makes CV useful when comparing distributions whose raw score scales differ.

### Simple example

Suppose one retrieval system produces scores around $0.8$ with SD $0.08$, while another produces scores around $8$ with SD $0.8$.

Their raw spreads are very different, but both have:

$$
CV = 0.10
$$

So CV says their spread is 10% of the mean in both cases.

That is why CV is being considered as a candidate dispersion signal for dense and sparse retrieval, rather than because CV is automatically the correct choice. The research must test this assumption.

## 9. Zero-Shot and Unsupervised

**Zero-shot** means applying a method to a query without training the method specifically for that query or target dataset.

**Unsupervised** means the method does not learn from labelled relevance judgments.

They are related but not identical. A method can be unsupervised but still use manually tuned parameters, for example.

For this research, the desired direction is a lightweight method that can adapt using information already present in the current retrieval results rather than requiring a labelled training pipeline.

## 10. Top-K

**Top-K** is the number of highest-ranked documents retained after retrieval.

For example, Top-5 means only the five highest-ranked results are passed to the next stage.

Adaptive Top-K changes this number depending on the query or retrieval conditions. It changes **how many documents are kept**, not the RRF rank-decay formula itself.

## 11. The Research Trade-off

The central trade-off is:

- **Score fusion:** preserves magnitude information but requires score compatibility and tuning.
- **RRF:** avoids score compatibility problems but discards magnitude information.
- **Dynamic RRF:** investigates whether some information from the score distribution can influence the rank-decay parameter without abandoning rank-based fusion.

This is a research hypothesis, not an established result. The experiments must determine whether the proposed signal actually improves retrieval.
