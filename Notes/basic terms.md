# Retrieval Fusion: Basic Terms

## 1. Linear Score Fusion

Linear score fusion combines the **raw scores** produced by different retrievers using weights.

$$
S(d) = \alpha S_1(d) + (1-\alpha)S_2(d)
$$

The main advantage is that it preserves score magnitude: a large confidence gap can influence the final score more strongly than a small gap.

The main difficulty is that different retrievers can produce scores on incompatible scales. Their scores may therefore need normalization, and the fusion weight may need tuning.

**Real-life analogy:** Imagine two teachers grading the same student. One gives marks out of 100 and another gives marks out of 10. Adding the raw marks would be meaningless unless their scales are aligned first.

## 2. Reciprocal Rank Fusion (RRF)

RRF ignores raw score magnitude and combines the **rank positions** assigned by different retrievers.

$$
\mathrm{RRF}(d) = \sum_{m \in M} \frac{1}{k + \mathrm{rank}_m(d)}
$$

where $M$ is the set of retrieval systems, $\mathrm{rank}_m(d)$ is the rank of document $d$ in system $m$, and $k$ is the smoothing constant.

The standard formulation traditionally uses $k = 60$.

### Why RRF is useful

Because it uses ranks rather than raw scores, RRF does not require the score scales of different retrievers to be aligned. This makes it particularly convenient for hybrid retrieval involving sparse and dense systems.

**Real-life analogy:** Two shopping websites rank the same products differently. Instead of comparing their internal recommendation scores, you simply look at where each product appears in each list and reward products that consistently rank near the top.

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

With a small $k$, A receives a stronger advantage from its rank-1 position. With a larger $k$, B's repeated mid-level appearances become relatively more competitive.

The exact effect depends on all retrieval lists, so $k$ should be understood as controlling the **shape of rank decay**, not as a direct measure of document relevance.

## 4. Static vs Dynamic $k$

A **static $k$** uses the same value for every query.

A **dynamic $k$** changes according to information available for the current query or retrieval result.

**Real-life analogy:** A car's automatic transmission changes gears depending on the road and speed. A fixed gear can work, but changing it according to the current conditions can be more responsive. Dynamic $k$ follows the same idea: the rank-decay setting may change according to the current retrieval situation.

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

**Real-life analogy:** A search engine may treat the first result as much more important than the tenth. Increasing $k$ is like making that preference less extreme.

## 6. Score Distribution

A **score distribution** describes how retrieval scores are spread across the returned documents.

For example:

- Scores such as $0.99, 0.98, 0.97, 0.96$ form a relatively tight group.
- Scores such as $0.99, 0.70, 0.40, 0.15$ show a much sharper drop.

The first pattern is a **plateau**: many candidates have similar scores. The second is a **cliff**: one or a few candidates stand clearly above the rest.

RRF cannot see this difference because both lists can have exactly the same rank positions.

**Real-life example:** Suppose a search for "best laptop for programming" gives five results with nearly identical relevance scores. The retriever is not finding a clear winner. A different query might produce one result far above all others. Score distribution exposes that difference; rank alone does not.

## 7. Score Dispersion

**Score dispersion** means how widely the scores are spread around the center of the distribution. In simple terms, it asks: **Are the scores tightly packed or widely separated?**

High dispersion can look like:

$$
0.99,\ 0.70,\ 0.40,\ 0.15
$$

Low dispersion can look like:

$$
0.99,\ 0.98,\ 0.97,\ 0.96
$$

Dispersion gives information about score gaps that rank positions throw away.

### Important caution

High dispersion does **not** mean high relevance. A model can be very confident and still be wrong. Therefore, dispersion is treated as a **heuristic signal** rather than a direct measure of truth or relevance.

**Real-life analogy:** A student may score 98, 70, 40, 15 on four topics. The large spread tells you their performance is uneven; it does not tell you whether the grading itself was correct.

## 8. Variance and Standard Deviation

**Variance** measures how far scores spread from their average, using squared differences.

**Standard deviation (SD)** is the square root of variance and expresses the spread in the same units as the scores.

A larger SD means the scores are more dispersed; a smaller SD means they are more tightly clustered.

These are descriptive statistics, not measures of retrieval relevance by themselves. In this research they are candidate signals that may describe the local shape of a retrieval score list.

**Simple example:** Scores $10, 10, 10, 10$ have almost no spread, while $2, 6, 10, 14$ have much more spread.

## 9. Coefficient of Variation (CV)

The **Coefficient of Variation (CV)** measures spread relative to the mean:

$$
CV = \frac{\sigma}{\mu}
$$

where $\sigma$ is the standard deviation and $\mu$ is the mean.

This can help compare relative spread when raw score scales differ.

### Simple example

Suppose one retrieval system produces scores around $0.8$ with SD $0.08$, while another produces scores around $8$ with SD $0.8$.

Their raw spreads are very different, but both have:

$$
CV = 0.10
$$

So CV says their spread is 10% of their mean in both cases.

That is why CV is being considered as a candidate dispersion signal for dense and sparse retrieval, rather than because CV is automatically the correct choice. The research must test this assumption.

## 10. Interquartile Range (IQR)

**Interquartile Range (IQR)** measures the spread of the middle 50% of the scores:

$$
IQR = Q_3 - Q_1
$$

where $Q_1$ is the 25th percentile and $Q_3$ is the 75th percentile.

IQR is often less affected by extreme outliers than variance or standard deviation.

**Why it matters here:** If one retrieval result has an unusually large score, IQR may describe the typical spread of the list more reliably than a measure that reacts strongly to that outlier.

## 11. Score Margin

A **score margin** is the difference between two scores, often the top two:

$$
\mathrm{Margin} = S_1 - S_2
$$

A large margin means the top result is far ahead of the runner-up. A small margin means they are close.

**Strength:** Very simple and focused on the top of the ranking.

**Limitation:** It ignores what happens further down the list. Two retrieval runs could have the same top-two margin but very different score distributions in the remaining 50 documents.

## 12. Rank Agreement

**Rank agreement** measures how much different retrieval systems agree on which documents should appear near the top.

For example, if BM25 and dense retrieval both put the same documents in their top 10, they have high agreement. If their top 10 lists barely overlap, they have low agreement.

Rank agreement is scale-free, but it describes **agreement between systems**, not the internal score distribution of one system.

## 13. Zero-Shot and Unsupervised

**Zero-shot** means applying a method to a query without training the method specifically for that query or target dataset.

**Unsupervised** means the method does not learn from labelled relevance judgments.

They are related but not identical. A method can be unsupervised but still use manually tuned parameters.

For this research, the desired direction is a lightweight method that can adapt using information already present in the current retrieval results rather than requiring a labelled training pipeline.

## 14. Top-K

**Top-K** is the number of highest-ranked documents retained after retrieval.

For example, Top-5 means only the five highest-ranked results are passed to the next stage.

Adaptive Top-K changes this number depending on the query or retrieval conditions. It changes **how many documents are kept**, not the RRF rank-decay formula itself.

## 15. Outlier

An **outlier** is a value that is unusually far from the other observations in a dataset.

In retrieval, an outlier could be one document with an unusually high score compared with the rest of the retrieved candidates.

Outliers matter because some dispersion measures, especially variance and standard deviation, can change substantially because of one extreme value.

## 16. Candidate Pool

The **candidate pool** is the collection of documents returned by a retrieval system before later stages such as fusion or reranking.

For example, if dense retrieval returns 100 documents and BM25 returns 100 documents, each 100-document list is a candidate pool for its respective modality.

Statistics such as mean, variance, and CV depend on which candidates are included, so changing the candidate-pool size can change the measured distribution.

## 17. BEIR

**BEIR** is a benchmark for evaluating information-retrieval systems across multiple datasets, tasks, and domains. It is useful here because our research should not be judged on one narrow collection alone.

**Why it matters:** A method that works only on one dataset may simply be fitting that dataset's retrieval behaviour. Testing several different datasets gives us a better idea of whether an observation is general.

**Practical note:** We do not need to run every available BEIR dataset. A small, deliberately chosen subset can be more realistic for an undergraduate experiment, provided the subset covers meaningfully different retrieval conditions.

## 18. NDCG and NDCG@10

**NDCG (Normalized Discounted Cumulative Gain)** measures how good a ranked list is by considering both relevance and position. A highly relevant document is worth more when it appears near the top than when it appears near the bottom.

The practical question is: **Did we put the useful documents where the user will see them first?**

### Real-life example

Suppose two search systems both find the same three useful papers.

- System A puts them at ranks 1, 2, and 3.
- System B puts them at ranks 8, 9, and 10.

Both systems found the papers, but System A produced a much better ranking. NDCG captures that difference.

**NDCG@10** means we evaluate only the top 10 results. This keeps the metric focused on the part of the ranking we care about most for retrieval evaluation.

### Limitation

NDCG is most informative when the dataset provides graded relevance labels. With only binary relevance labels, it still measures ranking quality, but some of its graded-relevance advantage is lost.

## 19. MRR

**Mean Reciprocal Rank (MRR)** focuses on the position of the **first relevant result**.

For one query:

$$
RR = \frac{1}{\mathrm{rank\ of\ first\ relevant\ result}}
$$

MRR is the average of this value over all queries.

### Real-life example

If the first useful result appears at rank 1, then:

$$
RR = 1
$$

If it appears at rank 5:

$$
RR = 0.2
$$

So MRR answers a very direct question: **How quickly does the system surface at least one useful result?**

### Why use it with NDCG?

MRR and NDCG measure different things. MRR mainly cares about the first useful result. NDCG cares about the quality and order of the broader ranked set.

Using both can reveal effects that one metric alone might hide. A method might move one relevant document to rank 1 while making the rest of the top 10 worse, or it might improve the whole top 10 without changing the first hit much.

### Limitation

MRR does not reward additional relevant documents after the first one. That makes it less suitable as a standalone measure for tasks that need several pieces of evidence.

## 20. Pre-computed Retrieval Runs

A **pre-computed retrieval run** is a saved list of documents and scores produced earlier by a retrieval system.

For example:

```text
Query 1 → BM25 → saved ranked list
Query 1 → Dense retriever → saved ranked list
```

We can load these saved lists and test different fusion algorithms without running BM25, embedding models, or vector databases again.

**Why this matters:** Our central research question concerns the **fusion step**. Using pre-computed runs lets us isolate that step and greatly reduce implementation work.

## 21. CPU-only Fusion Evaluation

The fusion experiment itself mainly performs numerical operations on already available retrieval results: ranking, arithmetic, and statistical calculations such as standard deviation.

A CPU is sufficient for this stage. This does **not** mean the entire retrieval pipeline is CPU-only. Dense embedding generation and dense retrieval can require substantial model computation.

### What we can claim

We can measure the CPU time required by the **fusion and dynamic-$k$ calculation** and report it as an additional systems metric.

We should not claim that the complete real-world hybrid retrieval pipeline is automatically cheap or CPU-only.

## 22. Baseline

A **baseline** is a reference method used for comparison.

For this research, standard RRF with fixed $k=60$ is the main baseline. We need it because an experimental method has no meaning unless we can show whether it improves over a known reference.

**Real-life analogy:** If you modify a car engine, you first compare it with the original engine under the same conditions. The original is the baseline.

## 23. Candidate Method

A **candidate method** is a possible approach that we are willing to test but have not yet accepted as the final method.

In this research, dispersion statistics such as CV, SD-based measures, or IQR are candidates. A candidate can be rejected after experiments.

This distinction is important: **being discussed in the research notes does not mean that it has been selected as the final algorithm.**

## 24. The Research Trade-off

The central trade-off is:

- **Score fusion:** preserves magnitude information but requires score compatibility and tuning.
- **RRF:** avoids score compatibility problems but discards magnitude information.
- **Dynamic RRF:** investigates whether information from the score distribution can influence the rank-decay parameter without abandoning rank-based fusion.

This is a research hypothesis, not an established result. The experiments must determine whether the proposed signal actually improves retrieval.
