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

This can outperform an untuned RRF baseline when good validation or training data is available. However, the learned weights are tied more closely to that data and may need to be retuned when the retrieval environment changes. Our research instead aims to investigate a lightweight, zero-shot control signal that comes directly from the current retrieval results.

## Our Solution

### Why Dynamic $k$?

The natural extension of the static baseline is to let $k$ change when the retrieval situation changes. A query-dependent $k$ means that each query can receive its own decay setting. A modality-dependent design goes one step further and allows values such as $k_{dense}$ and $k_{sparse}$ for the same query.

The key difference from grid search is that grid search chooses one value before deployment, while a dynamic method chooses the value at inference time. This could allow the system to react to the actual behaviour of the current retrieval lists.

This is still a hypothesis. A poorly designed dynamic rule could be less stable or less accurate than fixed $k=60$, so every dynamic method must be compared against the baseline.

### Why dispersion?

RRF knows the rank positions but ignores the numerical gaps between the scores that created those ranks. We therefore considered **score dispersion**, which simply asks how tightly or widely the retrieved scores are grouped.

For example, these two lists can have exactly the same ranks:

$$
0.99,\ 0.70,\ 0.40,\ 0.15
$$

and

$$
0.99,\ 0.98,\ 0.97,\ 0.96
$$

The first has a clear score separation, while the second is much flatter. RRF sees only rank 1, rank 2, rank 3, and rank 4. A dispersion statistic gives us one possible way to recover some of the information that rank-only fusion discarded.

However, dispersion is not a confidence score and is not proof that the top document is relevant. A model can be very confident and still be wrong. Therefore, dispersion is only a **heuristic signal** that needs experimental validation.

### Why compare Variance, SD, CV, and IQR?

We cannot choose the statistic just because it sounds intuitive. The literature we reviewed does not directly compare these four statistics for controlling the RRF smoothing constant. Choosing one as the winner before testing it would therefore be an unsupported assumption.

Variance and standard deviation are simple measures of absolute spread, but their values change when the score scale changes. This makes them difficult to compare directly between dense and sparse retrieval. IQR is much more resistant to outliers, but because it focuses on the middle 50% of the scores it may miss a sharp change near the top of the ranking.

CV is attractive because it measures spread relative to the mean. It can help compare relative spread when raw score scales differ, but it relies on the mean and can become unstable or hard to interpret when the mean is near zero or scores can be negative.

At this stage, we will **not select a final dispersion statistic**. CV and a normalized-SD alternative are candidate approaches to test, because the literature does not establish which statistic should control $k$. The experiment will decide whether either candidate is useful, whether another statistic performs better, or whether dispersion should be abandoned entirely.

### Why CV?

CV is a candidate, not our confirmed solution. We are considering it because it measures spread relative to the mean and may be more comparable across different score scales than raw variance or standard deviation.

The exact mapping from a dispersion statistic to $k$ is **not finalized**. We will first inspect the candidate statistics on real retrieval outputs and determine whether dispersion has a useful relationship with the choice of $k$ at all.

If CV is unstable or does not help retrieval, we will not force it into the final method. We can instead test another statistic, another mapping, or abandon the dispersion-based approach altogether.

### Why zero-shot?

The goal is to see whether the current retrieval results themselves contain enough information to choose a useful $k$. A zero-shot method does not need a labelled training pipeline for that decision.

This keeps the method lightweight and makes cross-dataset evaluation easier. The trade-off is that the method cannot learn a dataset-specific pattern from relevance labels, so it may not reach the peak accuracy of a well-trained fusion model.

### Why unsupervised?

We want the proposed control signal to come from the retrieval scores rather than from labelled relevance judgments. That keeps the method easier to reproduce and keeps the research focused on whether the retrieval distribution itself contains a useful signal.

Zero-shot and unsupervised are related but not identical. A method can be unsupervised and still contain manually tuned parameters. This distinction will matter when we evaluate any mapping parameters and safety rules in the final method.

## Evaluation

### Why BEIR?

BEIR is a benchmark designed to test information-retrieval systems across multiple tasks and domains. Its diversity is useful for this research because a candidate rule should not only work on one type of dataset.

We do not need to run the entire benchmark. A smaller set of clearly different datasets can make the experiment feasible while still testing whether the behaviour changes across domains and tasks. The exact subset should be finalized after checking the available pre-computed runs and the computational cost.

### Why NDCG?

NDCG measures both **which relevant documents are retrieved** and **where they appear in the ranking**. This matters because our research changes the order of documents, not just whether they are present.

For example, suppose two systems retrieve the same three relevant documents in their top 10. If one puts them at ranks 1, 2, and 3 while the other puts them at ranks 8, 9, and 10, NDCG gives the first system a much better score because the useful documents appear earlier.

We will use NDCG@10 as a candidate primary ranking metric because the top part of a retrieval list is especially important in retrieval evaluation. The exact cutoff should remain consistent across all methods.

### Why MRR?

MRR focuses on the position of the **first relevant result**. For one query, a relevant document at rank 1 gives a reciprocal rank of $1$, while a relevant document at rank 5 gives $0.2$.

This gives us a useful second perspective. NDCG@10 evaluates the quality of the whole top-ranked set, while MRR emphasizes how quickly the first useful result appears.

Using both helps us detect different effects of a dynamic fusion rule. A method could improve the first relevant result while making the rest of the top 10 worse, or it could improve the overall ranking without moving the first relevant result much.

### Why CPU-only?

The core research operation is the **fusion step**. If the dense and sparse retrieval runs are already available, calculating ranks, score statistics, and fusion scores involves ordinary numerical operations that can be performed on a CPU.

This does not mean the entire retrieval pipeline is CPU-only. Generating dense embeddings or producing dense retrieval runs may require substantial model computation. For our experiment, we can separate that stage from the fusion study by using pre-computed retrieval runs where available.

The CPU-only claim should therefore be limited to the fusion and dynamic-$k$ calculation itself. We can measure its actual execution time rather than assuming the overhead is negligible.

## Evaluation Plan

The experiment should remain an **isolated retrieval-fusion study**. We do not need to build a complete RAG application or generate LLM answers to test the central research question.

The basic pipeline is:

```text
Pre-computed dense run + pre-computed sparse run
                    ↓
              Fusion methods
                    ↓
              Ranked documents
                    ↓
             BEIR relevance data
                    ↓
             NDCG@10 + MRR
```

We should begin with a small, diverse subset of BEIR rather than committing to all datasets. The final subset should be chosen based on dataset diversity, availability of compatible runs, and practical runtime.

The first comparisons should include standard RRF with several fixed $k$ values so that the fixed-$k$ behaviour is understood before adding any dynamic rule. After that, candidate dynamic signals can be tested against the same inputs.

We should not decide in advance that CV, normalized SD, or any particular mapping function will be the final method. Those are experimental candidates.

Latency can be measured for the **fusion calculation itself** as an additional systems metric. An LLM generation stage, ROUGE/BLEU evaluation, or a complete vector-database deployment is not necessary for answering the core retrieval-fusion question.
