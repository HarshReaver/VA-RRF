# VA-RRF

**Working title:** Variance-Aware Reciprocal Rank Fusion for Hybrid Retrieval

## Research Question

Can the RRF smoothing constant be adapted per query and per retrieval modality using information about the score distribution, while preserving the simplicity and rank-based robustness of standard RRF?

## Current Research Direction

The study investigates a lightweight, zero-shot approach to hybrid retrieval that combines dense and sparse retrieval through Reciprocal Rank Fusion (RRF). The proposed direction is to investigate whether score-dispersion information can be used to adapt the fixed RRF smoothing constant dynamically.

## Repository Structure

```text
VA-RRF/
├── Decision Log/      # Final reasoning behind research decisions
├── Notes/             # Detailed paper analysis and learning notes
├── Papers/            # Literature metadata and bibliography
└── README.md
```

## Workflow

```text
Literature search
      ↓
Paper analysis
      ↓
Decision Log
      ↓
Method design
      ↓
Implementation
      ↓
Experiments
      ↓
Results
      ↓
Paper
```

The research decision log is the authoritative record of why each methodological choice is made.