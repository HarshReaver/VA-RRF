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

## Important limitation

Dynamic $k$ is only a research direction at this stage. It introduces a new mapping rule that could be unstable or could perform worse than the fixed $k=60$ baseline. The mapping must therefore be justified and tested rather than assumed to be beneficial.

## What remains unanswered

The next question is: **what measurable property of the current retrieval results should determine the dynamic $k$?**

Candidates to investigate include score dispersion measures such as variance, standard deviation, coefficient of variation, and IQR. These are candidates only; the literature review and experiments must determine whether any of them are appropriate.
