# Dynamic RRF $k$ Analysis

This note explains why we are investigating a query-dependent and modality-dependent RRF smoothing constant.

## What $k$ changes

RRF gives each document a contribution based on its rank:

$$
\mathrm{RRF}(d)=\sum_{m \in M}\frac{1}{k+\mathrm{rank}_m(d)}
$$

The smoothing constant $k$ controls the **shape of rank decay**. With a smaller $k$, the first few ranks matter much more. With a larger $k$, the decay becomes flatter and lower-ranked documents remain more competitive.

A simple real-life analogy is a recommendation list. A steep rule says, “being first matters much more than being fifth.” A flatter rule says, “several good positions should remain competitive.”

## Why One Fixed $k$ May Not Be Enough

A fixed $k$ applies the same decay curve to every query. But retrieval behaviour can change from one query to another. Dense and sparse retrieval may agree strongly on one query and disagree strongly on another. Their behaviour can also change when the test data comes from a different domain.

This does **not** prove that every query needs a different $k$. It gives us a reason to test whether one global value is always sufficient.

## The Existing Choices

The simplest choice is the established default $k=60$. It is useful because it gives us a clear and reproducible baseline.

Another choice is offline tuning. We can test several values of $k$ on a validation dataset and keep the one that performs best on average. This can improve results for that dataset, but the selected value remains fixed after deployment.

The difference is important:

```text
Offline tuning:
Many queries → find one good k → use the same k everywhere

Dynamic k:
Current query → inspect current retrieval behaviour → choose k
```

## Why Investigate Dynamic $k$?

A dynamic $k$ could respond to the current retrieval situation instead of forcing every query to use the same decay curve.

There are two related ideas. A **query-dependent $k$** chooses a value separately for each query. A **modality-dependent $k$** goes one step further and allows dense and sparse retrieval to use different values, such as $k_{dense}$ and $k_{sparse}$, for the same query.

This is the main difference from grid search. Grid search learns one global compromise before deployment. Dynamic $k$ changes at inference time.

## What Should Control $k$?

Once we decide that $k$ might change, we need a measurable signal that tells us when it should change.

One candidate is **score dispersion**, which means how tightly or widely the retrieved scores are grouped. For example:

$$
0.99,\ 0.70,\ 0.40,\ 0.15
$$

has a clear score separation, while:

$$
0.99,\ 0.98,\ 0.97,\ 0.96
$$

forms a much tighter group.

Both lists can have exactly the same rank positions. RRF sees rank 1, rank 2, rank 3, and rank 4 in both cases. Dispersion gives us another piece of information that rank alone does not contain.

However, a wide score spread does not prove that the top result is correct. A retriever can be confidently wrong. Dispersion is therefore a **heuristic signal**, not a direct relevance measure.

## Why This Fits the Research Goal

A dispersion statistic can be calculated from the scores that the retrievers already return. We do not need another neural model just to measure the shape of the score list.

That gives us a simple pipeline:

```text
Dense scores  → dispersion → k_dense ┐
                                     ├→ RRF fusion
Sparse scores → dispersion → k_sparse┘
```

The statistic only controls $k$. The final fusion remains rank-based.

This is attractive because it tries to recover some information lost by RRF without replacing RRF with a completely different fusion system.

## What Can Go Wrong?

A dynamic rule can fail in several ways. An outlier can make a score list look more dispersed than it normally is. Dense and sparse systems can have different score scales and shapes. Changing the number of retrieved candidates can also change the measured distribution. Finally, a sharp score separation can occur even when the retrieved documents are irrelevant.

These risks mean that the dynamic rule must be tested against the fixed $k=60$ baseline. We cannot assume that a more adaptive rule is automatically better.

## Current Research Question

The next question is: **which statistical measure should represent dispersion?**

Candidates are Variance, Standard Deviation, Coefficient of Variation (CV), and Interquartile Range (IQR). We should compare them before locking the final method.
