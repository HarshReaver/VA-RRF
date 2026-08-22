# Score Dispersion Analysis

This note explains why score dispersion is being considered as the signal for dynamic RRF $k$.

## Start With the Problem

RRF combines dense and sparse retrieval using rank positions. That is useful because the two retrievers can produce incompatible raw scores.

But there is a trade-off. RRF sees the order of the results and ignores the size of the gaps between their scores.

For example, these two retrieval lists have the same ranks:

$$
0.99,\ 0.70,\ 0.40,\ 0.15
$$

$$
0.99,\ 0.98,\ 0.97,\ 0.96
$$

The first list has a **score cliff**: the first result is far above the rest. The second has a **score plateau**: many candidates have similar scores.

RRF sees both lists as rank 1, rank 2, rank 3, and rank 4. It cannot see the difference between the two score patterns.

That gives us the basic idea behind dispersion: **can the spread of the current scores provide useful information for choosing the RRF smoothing constant $k$?**

## What Dispersion Tells Us

Score dispersion describes how tightly or widely the scores are grouped. A large spread means the retriever is separating some candidates from others more strongly. A small spread means many candidates have similar scores.

This does not mean that a large spread proves the top result is correct. A retriever can be very confident and still be wrong. Therefore, dispersion is only a **heuristic signal** (a practical signal used as a rule of thumb), not a direct relevance score.

### Real-life analogy

Imagine two exam classes. In Class A, the marks are 98, 97, 96, 95. The students are closely grouped. In Class B, the marks are 98, 70, 40, 15. One student is far ahead of the others.

The spread of marks tells us that the two classes have different score patterns. It does not tell us whether the exam itself was fair or whether the highest-scoring student truly understood the subject.

That is exactly how we should treat retrieval-score dispersion.

## Why Dispersion Fits Our Research Goal

We want to change only the RRF smoothing constant while keeping the final fusion rank-based.

A dispersion statistic is attractive because the retrievers already return scores. We do not need another neural model just to measure the shape of those scores.

The intended pipeline is:

```text
Query
  ↓
Dense retrieval → Top-N scores → dispersion statistic → k_dense
  ↓                                           │
                                               ├─→ RRF fusion
  ↑                                           │
Sparse retrieval → Top-N scores → dispersion statistic → k_sparse
```

The statistic is therefore used as a **control signal**. It is not inserted directly into the final RRF score.

## Other Signals We Could Use

Dispersion is not the only possible way to describe the retrieval results.

### Score Margin

The simplest alternative is the gap between the top two scores:

$$
\mathrm{Margin}=S_1-S_2
$$

A large margin means the top result is far ahead of the runner-up. A small margin means they are close.

The benefit is simplicity. The weakness is that it only looks at the top two scores. It cannot describe what happens across the rest of the candidate list.

### Rank Agreement

Another idea is to measure how much dense and sparse retrieval agree on the same documents near the top.

This is useful because it measures **agreement between retrievers**. The drawback is that it tells us about two lists together, whereas dispersion tells us about the internal score pattern of one list.

### Score Normalization

We could instead transform the scores so dense and sparse values become comparable before fusion. This keeps magnitude information, but it changes the fusion method itself and may introduce sensitivity to outliers and the chosen normalization rule.

### Neural Confidence

A neural model could judge the relevance of retrieved documents directly. That may be more powerful, but it adds model complexity, inference cost, and potentially training requirements. It also moves us away from the lightweight research question we are studying.

## Risks We Must Test

### Outliers

One unusually large or small score can make variance and standard deviation look larger than the typical spread of the list. A dynamic rule could then overreact to one strange result.

### Different Score Scales

Dense and sparse retrieval can use very different numerical scales. A statistic that looks large for BM25 may not represent the same relative behaviour as a similarly large value from dense retrieval.

### Candidate-Pool Size

The distribution depends on which candidates we include. If we calculate dispersion over Top-10 in one experiment and Top-100 in another, the resulting statistic may change even when the query is the same.

The experiment therefore needs a controlled Top-$N$ setting before comparing dispersion measures.

### False Confidence

A sharp score cliff can mean that the retriever strongly prefers one candidate. It does not guarantee that the candidate is actually relevant.

This is why the paper should describe dispersion as a **signal of score separation**, not as a measure of truth or confidence.

## What the Literature Says

The papers we reviewed do not directly compare Variance, Standard Deviation, CV, and IQR for the purpose of changing the RRF smoothing constant. That means the choice between these statistics is an empirical research question rather than an established result.

The literature does, however, provide a reason to investigate score-distribution information. Work discussing the limits of rank-only fusion points to the loss of score-magnitude information, while adaptive retrieval work shows that query-level score distributions can be useful for making retrieval decisions.

Our contribution is therefore not based on assuming that dispersion is already proven useful. The research question is whether a simple dispersion signal can make RRF more adaptive without sacrificing its practical advantages.

## Current Candidate Statistics

We will compare four common dispersion measures:

- **Variance:** measures absolute spread using squared differences from the mean.
- **Standard deviation:** expresses spread in the same units as the original scores.
- **Coefficient of Variation (CV):** measures spread relative to the mean.
- **Interquartile Range (IQR):** measures the spread of the middle 50% of the scores.

The current hypothesis favours CV because dense and sparse scores can have different scales. However, the literature does not justify declaring CV the winner before experiments.

The current experimental plan therefore carries CV and a normalized-SD alternative forward. The experiments will tell us whether the extra scale handling of CV actually improves stability and retrieval quality.

## Next Step

After choosing the statistic, we still need to justify the mapping from that statistic to $k$.

For example, our current candidate is:

$$
k_m=60e^{-\beta CV_m}
$$

The next question is not simply whether this formula looks elegant. We need to explain **why $k$ should move in this direction, what $\beta$ does, what can go wrong, and what alternative mapping we will use if the first mapping fails.**
