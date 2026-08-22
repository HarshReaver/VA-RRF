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

## 3. The Research Gap

The central trade-off is:

- **Score fusion:** preserves magnitude information but requires score compatibility and tuning.
- **RRF:** avoids score compatibility problems but discards magnitude information.

VA-RRF investigates whether information about the score distribution can be used to adapt RRF's smoothing constant while retaining its rank-based formulation.
