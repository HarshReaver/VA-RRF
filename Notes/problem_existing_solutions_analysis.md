# Problem and Existing Solutions Analysis

This note records the NotebookLM analysis used to build the `Problem` and `Existing Solutions` sections of the research decision log.

## 1. Why is static $k$ a problem?

In RRF, the smoothing constant $k$ controls how quickly the contribution of rank decreases.

$$
RRF(d)=\sum_{m \in M}\frac{1}{k+r_m(d)}
$$

A smaller $k$ makes rank differences more influential, so top-ranked documents receive a stronger advantage. A larger $k$ flattens the decay and makes lower ranks contribute more similarly to higher ranks.

The original RRF work selected $k=60$ empirically rather than deriving it from a theoretical optimum. Bruch et al. (2023) report that RRF is sensitive to its parameters, including $k$, and show that tuned alternatives can outperform untuned RRF in some settings.

The resulting concern is not that $k=60$ is always wrong. It is that one fixed value may not be equally suitable for different retrieval conditions.

## 2. Why not keep $k=60$?

$k=60$ remains an appropriate baseline because it is the established RRF default and provides a consistent reference point.

However, it was empirically selected and then kept fixed. Literature showing sensitivity to RRF parameters means that $k=60$ should not be assumed to be universally optimal.

Therefore, the experiment should retain standard RRF with $k=60$ as the primary baseline rather than replacing it with an allegedly superior fixed value.

## 3. Offline Grid Search

A common approach is to test several candidate values of $k$ on a validation set and select the value producing the best retrieval metric.

**Strength:** Simple and effective when a suitable validation set and relevance judgments are available.

**Limitation:** The selected value is usually a global setting optimized for the validation distribution. It does not naturally adapt to the characteristics of each query at inference time.

Thus, offline tuning can improve RRF for a known target distribution without providing per-query adaptation.

## 4. Score Normalization / DBSF

Score normalization attempts to make heterogeneous retriever scores comparable before fusion.

Min-Max normalization uses:

$$
S_{norm}=\frac{S-S_{min}}{S_{max}-S_{min}}
$$

Distribution-based methods use statistics of the retrieved score distribution to transform scores before combining them.

**Strength:** Score-based fusion retains magnitude information that RRF discards.

**Limitation:** The transformed result remains dependent on the shape of the score distribution, including the effect of outliers. It also operates directly on the raw-score space rather than adapting the rank-decay function inside RRF.

## 5. Neural Rerankers / Learned Approaches

Learned fusion methods learn how to combine retrieval signals, while neural rerankers examine retrieved candidates using a neural model or LLM and produce a new ranking.

**Strength:** These methods can capture deeper query-document relationships than simple rank-based heuristics.

**Limitation:** They introduce additional model inference cost, latency, implementation complexity, and in some cases training or task-specific data requirements.

They therefore address a broader semantic-ranking problem rather than specifically modifying the RRF rank-decay mechanism.

## Overall Problem Chain

```text
RRF uses a fixed k
        ↓
k controls rank contribution and decay
        ↓
Different retrieval conditions may favor different decay behavior
        ↓
Existing approaches include offline k tuning,
score normalization, and neural/learned reranking
        ↓
These approaches involve either static tuning,
raw-score transformation, or additional model complexity
        ↓
Research question:
Can RRF's rank-decay behavior be adapted at inference time
using information already present in the current retrieval results?
```

**Important distinction:** This note does not establish VA-RRF as novel. It only documents the problem and existing solution space that must be considered before evaluating the proposed method.
