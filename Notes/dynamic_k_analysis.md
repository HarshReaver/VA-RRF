# Dynamic RRF $k$ Analysis

This note records the literature-based reasoning behind investigating a query-dependent and modality-dependent RRF smoothing constant.

## What $k$ changes

RRF assigns each ranked document a contribution based on its rank:

$$
\mathrm{RRF}(d)=\sum_{m \in M}\frac{1}{k+\mathrm{rank}_m(d)}
$$

The smoothing constant $k$ controls the **shape of rank decay**.

- Small $k$: top ranks receive relatively more influence.
- Large $k$: rank contributions become flatter, so lower ranks remain more competitive.

A useful mental model is a recommendation list. With steep rank decay, being #1 matters much more than being #5. With flatter decay, several strong rankings across different lists can matter more collectively.

## Why one fixed $k$ may be insufficient

A fixed $k$ applies the same rank-decay behavior to every query. Retrieval conditions can vary across queries and domains: dense and sparse systems may agree strongly on some queries and disagree on others, and their effectiveness can change under domain shift.

This does not prove that every query needs a different $k$. It only provides a reason to investigate whether a single global value is always appropriate.

## Existing ways to choose $k$

### Default $k=60$

The standard RRF baseline uses $k=60$. It is useful because it is established and reproducible.

### Offline tuning

A researcher can evaluate candidate values of $k$ on a validation dataset and select the best one according to a retrieval metric. This can improve average performance for the target distribution, but the selected value remains fixed during inference.

## Why investigate dynamic $k$

A dynamic $k$ would allow the fusion rule to respond to the current retrieval situation rather than using the same decay curve for every query.

There are two distinct ideas:

- **Query-dependent $k$:** choose $k$ separately for each query.
- **Modality-dependent $k$:** allow dense and sparse retrieval to use different $k$ values for the same query.

These are different from offline global tuning because the value can change at inference time.

## What should drive dynamic $k$?

The next design question is what measurable property of the current retrieval results should control $k$.

One candidate is **score dispersion**: how tightly or widely the returned scores are grouped. For example:

- $0.99, 0.70, 0.40, 0.15$ shows a sharp separation.
- $0.99, 0.98, 0.97, 0.96$ shows a tight cluster.

This information is not present in ranks alone. A rank-based method sees both examples simply as rank 1, rank 2, rank 3, and rank 4.

The literature provides a reasonable basis for investigating score distributions as a query-dependent signal, but this does **not** prove that dispersion is a direct measure of relevance. A retriever can be confidently wrong, especially under domain shift.

## Why dispersion is attractive

A dispersion statistic can be computed directly from the scores already returned by the retrieval system. It requires no neural model and can be used only to change $k$ while leaving the final fusion formula rank-based.

This preserves the main operational advantage of RRF while allowing the research to test whether some score-distribution information can reduce its magnitude blindness.

## Important risks

Dynamic dispersion-based control can fail in several ways:

- **Outliers:** One extreme score can make the distribution appear more dispersed than it really is.
- **Modality differences:** Dense and sparse scores can have very different numerical scales and shapes.
- **Candidate-pool effects:** Statistics can change when the number of retrieved candidates changes.
- **False confidence:** A sharp score separation can occur even when the top documents are irrelevant.

Therefore, dispersion must be treated as a **heuristic signal**, not as a direct relevance score.

## What remains unanswered

The next question is which dispersion statistic is most appropriate for the two retrieval modalities.

Candidates include:

- Variance
- Standard deviation
- Coefficient of Variation (CV)
- Interquartile Range (IQR)

The research must compare them before selecting one for the proposed method.
