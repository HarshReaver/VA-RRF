# 1. Linear Score Fusion

### Combines the actual raw scores (points) using custom weights.

#### Final Score = (Weight A × Score A) +.... (Weight N × Score N)

### Big Pro: Preserves exact score strengths and gives you precise control.

### Big Con: Hard to set up because you must first fix (normalize) completely different scoring scales.

# 2. Reciprocal Rank Fusion (RRF)

### It ignores scores completely and looks only at the order (rank). It adds up fractions like 1/1, 1/2, or 1/3, making it much easier to use when completely different scoring systems are involved.

$$
\text{RRF Score}(d) = \sum_{m \in M} \frac{1}{k + \text{rank}_m(d)}
$$

#### Uses fractions based on placement: 1 / (60 + Rank)

### Big Pro: Simple, reliable, and works instantly without fixing any score scales.

### Big Con: Throws away detailed score data (treats a close win and a massive win the same).
