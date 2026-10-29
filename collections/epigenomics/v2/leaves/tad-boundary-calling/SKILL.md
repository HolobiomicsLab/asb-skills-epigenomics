---
name: tad-boundary-calling
description: Use when you need to call TAD boundaries from an NxN intra-chromosomal
  Hi-C contact matrix (e.g., GM12878 at 50 Kb/25 Kb/5 Kb or simulated IMR-90 matrices)
  without manual parameter tuning.
when_to_use_negative:
- Input is raw .hic/.cool data that has not yet been dumped to an NxN contact matrix
  — TOAST uses the NxN format as input, so pre-dump with Juicer tools first
- You need parameter-free benchmarking against methods requiring metric configuration
  (e.g., TopDom) as the comparison target rather than TAD calling itself
- Regions shorter than 200 Kb must be reported — contiguous same-label regions ≤200
  Kb are filtered out by design
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3957
tools:
- name: TOAST
  role: 'End-to-end TAD identification: preprocessing, GAE training, clustering, TAD
    generation (toast.py)'
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Unsupervised clustering of the 16-dimensional GAE embeddings to label each
    genomic bin
- name: TopDom
  role: Baseline TAD caller for FMS/TAD-quality comparison on simulated matrices
  repo: https://github.com/ghaiyan/TOAST
- name: HiCSeg
  role: Baseline statistical TAD caller for FMS/TAD-quality comparison
  repo: https://github.com/ghaiyan/TOAST
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:e394e0cd2e4d2d50329cb5e1e8af56365406fa762c796b2e7df39181d8c5fbb0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - HDBSCAN
  - HiCSeg
  - TOAST
  - TopDom
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tad-boundary-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

TOAST identifies topologically associated domain (TAD) boundaries by treating each genomic bin as a graph node with the Hi-C contact matrix as adjacency matrix, training a 2-layer graph auto-encoder (hidden_dim=64, embedding_dim=16) on non-overlapping 400x400 submatrices, clustering the 16-dimensional embeddings with HDBSCAN, and retaining contiguous same-label regions longer than 200 Kb as TADs.

## When to use

Use when you need to call TAD boundaries from an NxN intra-chromosomal Hi-C contact matrix (e.g., GM12878 at 50 Kb/25 Kb/5 Kb or simulated IMR-90 matrices) without manual parameter tuning.

## When NOT to use

- Input is raw .hic/.cool data that has not yet been dumped to an NxN contact matrix — TOAST uses the NxN format as input, so pre-dump with Juicer tools first
- You need parameter-free benchmarking against methods requiring metric configuration (e.g., TopDom) as the comparison target rather than TAD calling itself
- Regions shorter than 200 Kb must be reported — contiguous same-label regions ≤200 Kb are filtered out by design

## Inputs

- NxN tab-separated intra-chromosomal Hi-C contact matrix (KR-normalized, dumped from .hic via Juicer tools)
- Genomic bin size (e.g., 50 Kb) to set the 400x400 submatrix partition and 200 Kb length threshold
- Simulated Hi-C contact matrices (Wang et al. IMR-90 noise levels 4-20; Forcato et al.; Trussart et al.) for benchmarking

## Outputs

- TAD boundary file in (start bin, start location, end bin, end location) format
- TAD heatmap plots (plot_heatmaps.py)
- TAD_quality and FMS evaluation tables (e.g., simulated_benchmark_table.csv)

## How to apply

Dump the KR-normalized intra-chromosomal contact matrix at the chosen resolution (e.g., 50 Kb from .hic accession 4DNFI1UEG1HD using Juicer tools) as an NxN matrix. Partition each chromosome's matrix along the diagonal into non-overlapping 400x400 submatrices (n=400 at 50 Kb resolution) and train a 2-layer graph auto-encoder (hidden layer 64, embedding 16, lr=0.001, 200 epochs, Adam, MSE reconstruction loss) per submatrix to obtain 16-dimensional node embeddings. Cluster embeddings with HDBSCAN to label every genomic bin, then group contiguous same-label bins and keep only regions exceeding 200 Kb (at least 4 bins at 50 Kb) as TADs, written as (start bin, start location, end bin, end location). The hidden_dim=64/output_dim=16 configuration was selected as most robust over a one-layer network with output_dim=128. Validate TADs by checking TAD_quality (intra-TAD minus inter-TAD average contact frequency), a rational TAD count, and average TAD length within the expected 200 Kb-5 Mb interval.

## Related tools

- **TOAST** (End-to-end TAD identification: preprocessing, GAE training, clustering, TAD generation (toast.py)) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Unsupervised clustering of the 16-dimensional GAE embeddings to label each genomic bin)
- **TopDom** (Baseline TAD caller for FMS/TAD-quality comparison on simulated matrices) — https://github.com/ghaiyan/TOAST
- **HiCSeg** (Baseline statistical TAD caller for FMS/TAD-quality comparison) — https://github.com/ghaiyan/TOAST

## Examples

```
python toast.py  # in /src, after changing the input and output folder paths (input: NxN contact matrix; output: merged TAD file)
```

## Evaluation signals

- On Wang et al. simulated data (noise levels 4-20), FMS should remain around 0.98 (≈0.99 at noise 4-8, ≈0.975-0.981 at noise 12-20); TAD count vs ground truth differs by only 1-8 and avg length by up to 5 Kb
- TAD_quality = average intra-TAD minus inter-TAD contact frequency should be a rational positive number (e.g., 1654.03 on GM12878 50 Kb; >10 on simulated noise 16)
- Called TADs should average between 200 Kb and 5 Mb in length; at 50 Kb resolution each retained TAD must span ≥4 bins
- All chromosomes 1-22 processed; on GM12878 50 Kb the average TAD number (~4736) should surpass other algorithms such as TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, CASPIAN
- TAD boundary anchors should show expected ChIP-seq enrichment (e.g., average CTCF anchoring ratio 0.66, SMC3 0.47, RAD21 0.54)

## Limitations

- Existing TAD identification approaches (and by extension the comparison setting) can be impaired by complex parameter settings, lengthy computation time, or low accuracy
- The GAE is trained per non-overlapping 400x400 submatrix, so results depend on the resolution-dependent submatrix size (n=400 at 50 Kb); other resolutions need re-checking
- Contact matrices must be provided in NxN format; .hic files require a separate Juicer tools dump step
- Performance was validated on GM12878 (real) and IMR-90/Trussart/Forcato (simulated) datasets; other cell types or very low-coverage data are untested in this context

## Evidence
