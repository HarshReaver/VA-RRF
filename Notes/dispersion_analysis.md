# Score Dispersion Analysis

This note records the literature-based reasoning for considering score dispersion as a possible signal for adapting the RRF smoothing constant $k$.

## What Score Dispersion Means

Score dispersion describes how tightly or widely the retrieved scores are distributed for one query.

A list such as:

$$
0.99,\ 0.70,\ 0.40,\ 0.15
$$

has a much sharper separation than:

$$
0.99,\ 0.98,\ 0.97,\ 0.96
$$

The first can be described as a **score cliff** and the second as a **score plateau**.

### Why this matters to RRF

RRF sees only rank positions. Both lists can have the same ranks: 1, 2, 3, 4. The numerical gaps between those ranks are invisible to RRF.

Therefore, dispersion potentially provides information that a rank-only method does not use.

## What Dispersion Could Tell Us

A broad score spread may indicate that the retriever is separating a small number of candidates from the rest. A narrow spread may indicate that many candidates are being scored similarly.

However, this should be treated as an **inference about retrieval behavior**, not as proof of relevance.

A retriever can be confidently wrong, especially when the query is outside the model's familiar domain. Therefore:

> High dispersion means strong separation in the retriever's scores, not necessarily high correctness.

## Evidence From the Literature

The uploaded literature provides support for investigating score distributions as a query-dependent signal. Adaptive retrieval work uses similarity-score distributions to make retrieval decisions, while work discussing the limits of rank-only fusion highlights the loss of score-magnitude information.

The key point is narrower than saying that dispersion is already proven to be a relevance measure: the literature gives a reason to **investigate** whether score-distribution information can help control retrieval behavior.

## Alternative Signals

Before selecting a dispersion statistic, several other signals should be considered.

### Score Margin

The top-two score margin is:

$$
\mathrm{Margin}=S_1-S_2
$$

It captures whether the top result clearly separates from the runner-up.

**Advantage:** Very simple and focused on the top of the list.

**Limitation:** It ignores the rest of the candidate pool.

### Rank Agreement

Rank agreement measures how much two retrieval systems agree on the same documents and ordering, especially near the top.

**Advantage:** Scale-free and directly measures cross-retriever consensus.

**Limitation:** It does not describe the internal score distribution of one retriever.

### Score Normalization

Score normalization transforms raw scores before fusion so that different retrievers become more comparable.

**Advantage:** Preserves score magnitudes for fusion.

**Limitation:** It changes the score-fusion mechanism itself and can remain sensitive to score-distribution problems and outliers.

### Neural Confidence / Reflection

A neural model can explicitly evaluate whether retrieved passages are relevant.

**Advantage:** Potentially much richer semantic assessment.

**Limitation:** Higher computational cost, latency, and model complexity.

## Why Dispersion Is Attractive

Dispersion is attractive because it can be calculated directly from the scores already produced by the retriever.

That means it can potentially be used at inference time without:

- a neural reranker
- labelled training data
- an additional model
- a GPU-heavy pipeline

The intended design is to use the statistic only as a control signal for $k$, while retaining RRF's rank-based fusion formula.

## Risks

### Outliers

A single extreme score can increase standard deviation or variance and make a list look more dispersed than its typical values suggest.

### Modality Differences

Dense and sparse retrieval systems may produce very different numerical score scales and distributions. A dispersion statistic therefore needs to be interpreted carefully for each modality.

### Candidate-Pool Dependence

The measured distribution can change when the number of retrieved candidates changes. The experiment must therefore keep the candidate-pool size controlled when comparing methods.

### False Confidence

A clear score separation can occur even when the retrieved documents are wrong. Dispersion must therefore remain a heuristic signal.

## Decision

There is a reasonable literature-based justification for investigating score dispersion as a signal for adapting RRF $k$.

The literature does **not** establish that dispersion is a direct relevance measure or that any particular dispersion statistic will produce the correct dynamic $k$.

Those are empirical questions for the next stage of the research.

## Next Question

Which dispersion statistic should be tested first for dense and sparse retrieval?

Candidates:

- Variance
- Standard deviation
- Coefficient of Variation (CV)
- Interquartile Range (IQR)
