# Research Decision Log

This file records the reasoning behind the research. It should answer one simple question at every stage: **why are we doing this?** Detailed paper analysis stays in `Notes/`; this file keeps the final reasoning readable and connected.

## Research flow

The project starts with a RAG system that needs to retrieve useful information before an LLM answers a question. We use **hybrid retrieval**, which combines two retrieval modalities: dense retrieval and sparse retrieval. Dense retrieval is mainly used for semantic matching, while sparse retrieval such as BM25 is mainly used for exact keyword matching.

The two retrieval lists then need to be merged. RRF is a common choice because it works with ranks rather than raw scores. Its basic idea is:

$$
\mathrm{RRF}(d)=\sum_{m \in M}\frac{1}{k+\mathrm{rank}_m(d)}
$$

The problem is that the same smoothing constant $k$ is normally used for every query and every modality. Our research asks whether information that RRF currently ignores, especially the shape of the retrieved score distribution, can be used to adapt $k$ without giving up the simplicity of rank-based fusion.

A simple mental model is:

```text
Query
  ↓
Dense retrieval ──→ semantic matches
  │
  ├──→ RRF fusion ──→ final ranking
  │
Sparse retrieval ─→ keyword matches

RRF currently uses the ranks.
Our research asks whether the score distribution can also help choose k.
```

## Retrieval

### Why Hybrid Retrieval?

We use both dense and sparse retrieval because they find different kinds of matches. Dense retrieval can find documents that are semantically related even when the exact words are different. Sparse retrieval is strong when the important words appear directly in the document. Using both gives the system two complementary ways to find useful documents.

Using only dense retrieval risks missing exact terminology. Using only sparse retrieval risks missing synonyms and semantic relationships. The trade-off is that the two systems produce different kinds of scores, so simply adding their raw scores is not straightforward.

### Why BM25?

BM25 is our sparse retrieval baseline because it is well established, fast, requires no training, and is easy to reproduce. It is also simple enough that the behaviour of the sparse side of the experiment remains easy to explain.

The main weakness is that BM25 depends on lexical overlap. If a relevant document uses different words from the query, BM25 may miss it. That weakness is useful for our research because dense retrieval can complement it.

### Why Dense Retrieval?

We include a dense retriever because it can represent the meaning of a query and a document in a vector space. This allows it to find semantically related text even when the wording is different.

Dense retrieval is not perfect. It can struggle when the data is far from the data used to build or train the retriever. This is another reason to combine it with a lexical system rather than relying on one retrieval method.

### Why RRF?

Once dense and sparse retrieval produce two ranked lists, we need a way to merge them. Linear score fusion is tempting because it keeps score magnitude, but dense and sparse scores are usually on different scales. That means the scores need to be normalized or weighted carefully.

RRF avoids that problem by ignoring the raw scores and using only the ranks. This makes the method simple, deterministic, and easy to reproduce across different retrievers.

The cost of that simplicity is **magnitude blindness**. For example, a dense retriever might produce scores of $0.99$ and $0.50$, while another query produces $0.99$ and $0.98$. If the rank order is the same, RRF treats both situations identically because it cannot see the size of the score gap.

So the first research clue is simple: **RRF is robust because it ignores score magnitude, but that same choice may cause it to ignore useful information.**

## Problem

### Why is static $k$ a problem?

The smoothing constant $k$ controls how strongly rank affects the RRF contribution. A small $k$ makes the decay steeper, so the first few ranks matter much more. A large $k$ makes the decay flatter, so lower-ranked results remain more competitive.

The issue is not that $k=60$ is mathematically wrong. The issue is that the same decay curve is applied to every query. Two queries can produce very different retrieval behaviour: one may have a clear top result, while another may have many similarly scored candidates. Using exactly the same $k$ for both may not always be ideal.

The literature also reports sensitivity to RRF parameters. This gives us a reason to investigate whether the decay should respond to the current retrieval situation rather than always using one global value.

### Why not keep $k=60$?

We will keep $k=60$ as the main baseline. It is established, reproducible, and gives us a fair reference point. We should never present standard RRF as a bad method merely because we are proposing a modification.

The research question is narrower: **can we improve on the fixed baseline by changing $k$ only when the current retrieval results provide a reason to do so?**

## Existing Solutions

### Offline Grid Search

One straightforward approach is to try many values of $k$ on a validation dataset and keep the one that gives the best average retrieval score. This is useful because it can improve RRF for a known dataset without changing the algorithm itself.

Its limitation is that the chosen value is still global. A value that is good on average may not be the best value for every individual query. Offline tuning also needs evaluation data and does not react to a new query at inference time.

### Score Normalization (DBSF)

Another approach is to keep the raw scores and make them comparable before combining them. Methods such as Min-Max normalization or DBSF try to put different score distributions into a common space.

This keeps information about score gaps, which RRF throws away. The trade-off is that the fusion now depends on how well the score transformation handles the underlying distributions and outliers. It also changes the fusion mechanism rather than adapting the RRF smoothing constant itself.

### Neural Rerankers

A neural reranker takes the retrieved candidates and uses a stronger model to judge query-document relevance again. This can improve ranking quality because the model can examine the text more deeply than a simple rank-fusion rule.

The price is additional model inference, latency, implementation complexity, and sometimes training data. Neural reranking is therefore a different solution to a broader ranking problem rather than a lightweight change to RRF itself.

### Adaptive Top-K

Adaptive Top-K changes how many retrieved documents are kept for the next stage. For a simple query, it might keep fewer documents; for a difficult query, it might keep more.

This solves a **context-quantity** problem. Dynamic RRF $k$ solves a different problem: it changes how strongly ranks are weighted while the retrieval lists are being fused. Adaptive Top-K does not repair a bad ranking before the cutoff.

### Learned Fusion Weights

Learned fusion weights change how much each retrieval channel contributes to the final ranking. For example, a system may learn to trust dense retrieval more than sparse retrieval for a particular dataset.

This can outperform an untuned RRF baseline when good validation or training data is available. However, the learned weights are tied more closely to that data and may need to be retuned when the retrieval environment changes.

Our research is considering a different direction: a lightweight, zero-shot rule that could use information from the current retrieval results themselves. This remains a research hypothesis and still needs to be tested.

## Our Solution

### Why Dynamic $k$?

A natural question is whether $k$ should always stay fixed. A query-dependent $k$ would let different queries use different decay settings, while a modality-dependent design would allow dense and sparse retrieval to use different values for the same query.

This is different from offline grid search. Grid search chooses one value before deployment; a dynamic method would choose the value at inference time using the current retrieval situation.

At this stage, **dynamic $k$ is a research direction, not a confirmed improvement**. We still need to determine whether it helps at all, what information should control it, and whether it can beat the fixed $k=60$ baseline consistently.

### Why dispersion?

One possible source of information is **score dispersion**, which means how tightly or widely the retrieved scores are grouped.

For example, these two lists can have exactly the same ranks:

$$
0.99,\ 0.70,\ 0.40,\ 0.15
$$

and

$$
0.99,\ 0.98,\ 0.97,\ 0.96
$$

The first has a clear score separation, while the second is much flatter. RRF sees only rank 1, rank 2, rank 3, and rank 4. A dispersion statistic could therefore provide information that rank-only fusion currently ignores.

However, this is only a **candidate hypothesis**. Dispersion is not a direct measure of relevance. A retriever can produce a sharply separated list and still be wrong. Outliers, modality differences, and candidate-pool size can also distort the statistic.

So the current question is not "how do we use dispersion?" but **"does dispersion contain a useful signal for choosing $k$?"** That must be established experimentally.

### Why compare Variance, SD, CV, and IQR?

We should not select one statistic before testing it. The literature we reviewed does not directly establish that Variance, Standard Deviation, Coefficient of Variation (CV), or Interquartile Range (IQR) is the correct signal for controlling the RRF smoothing constant.

Variance and standard deviation measure absolute spread, so their values depend on the numerical scale of the scores. IQR is more resistant to outliers but mainly describes the middle half of the distribution. CV measures spread relative to the mean, which may make it useful when comparing different score scales, but it can become unstable when the mean is near zero and requires care with negative scores.

Therefore, **no dispersion statistic is selected yet**. These metrics are candidates for experiments. We should compare their behaviour on real dense and sparse retrieval outputs before deciding whether any of them belongs in the final method.

### Why CV?

CV is one candidate statistic, not our confirmed solution.

It is interesting because it measures spread relative to the mean, which can make it less dependent on the raw units of the scores than variance or standard deviation. That makes it worth testing for dense and sparse retrieval, which can produce different score scales.

But we have not established that CV is stable, predictive of a useful $k$, or better than other statistics. We also have not chosen a final mapping from a statistic to $k$.

The earlier expression

$$
k_m = 60e^{-\beta CV_m}
$$

is therefore **only an example candidate mapping**, not the committed algorithm. We may test it, replace it with another mapping, use another statistic, or abandon dispersion entirely depending on the experimental results.

### Why zero-shot?

A zero-shot approach is attractive because it would use the current retrieval results without requiring a labelled training pipeline. That keeps the method lightweight and makes it easier to test across different datasets.

However, zero-shot does not automatically mean better. A trained fusion model can learn dataset-specific patterns that a simple rule cannot. This is a trade-off that the experiments must measure.

### Why unsupervised?

The proposed control signal is being considered from the retrieval results themselves rather than from labelled relevance judgments. This keeps the method simpler and makes it easier to reproduce.

Zero-shot and unsupervised are related but not identical. An unsupervised method can still contain manually tuned parameters. We therefore need to be careful not to call the final method "zero-tuning" until the experimental design establishes that claim.

## Evaluation

### Why BEIR?

### Why NDCG?

### Why MRR?

### Why CPU-only?
