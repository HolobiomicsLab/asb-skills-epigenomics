---
name: hic-contact-map-downsampling
description: Use when you have replicated pairwise Hi-C contact maps (e.g., ESC vs
  NPC at 100 kb) with known full-depth dcHiC calls and need to determine the minimum
  sequencing depth at which differential compartments remain recoverable.
when_to_use_negative:
- Input consists of single-cell Hi-C maps without pseudo-bulk aggregation — the paper
  notes compartment identification is limited by sparsity of single-cell contact maps,
  making down-sampling recall curves ill-defined
- You lack a full-depth reference call set; recall cannot be computed without the
  full-depth differential bins
- Analysis target is differential loops or interactions (Fit-Hi-C output) rather than
  compartment calls — down-sampling effects on contacts are a different question
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0622
tools:
- name: dcHiC
  role: Rerun of the compartment pipeline (PCA of observed/expected-normalized correlation
    matrix, quantile normalization, Mahalanobis-distance outlier testing with chi-square
    significance and FDR10) on each down-sampled pairwise map set
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:b385c8ca8ddc837c1ba4c035a70f8b6e936f123214a4dc64c0e8036fc5ff5d9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hic-contact-map-downsampling

## Summary

Systematically down-samples Hi-C replicates to fractions of their original sequencing depth and re-runs dcHiC's differential compartment pipeline to quantify how many significant compartment calls are retained (recall) as depth decreases.

## When to use

Use when you have replicated pairwise Hi-C contact maps (e.g., ESC vs NPC at 100 kb) with known full-depth dcHiC calls and need to determine the minimum sequencing depth at which differential compartments remain recoverable.

## When NOT to use

- Input consists of single-cell Hi-C maps without pseudo-bulk aggregation — the paper notes compartment identification is limited by sparsity of single-cell contact maps, making down-sampling recall curves ill-defined
- You lack a full-depth reference call set; recall cannot be computed without the full-depth differential bins
- Analysis target is differential loops or interactions (Fit-Hi-C output) rather than compartment calls — down-sampling effects on contacts are a different question

## Inputs

- Full-depth Hi-C contact maps for replicated pairwise conditions (ESC: 4 replicates, 230M-1.2B reads; NPC: 3 replicates, 720M-1.5B reads) at 100 kb resolution
- Full-depth dcHiC differential compartment calls (reference bin set, e.g., 1981 significant bins; 1906 in the 3v3 comparison excluding the 230M-read ESC replicate)
- dcHiC filter/settings configuration (RefineY_Rconf90_FDR10)

## Outputs

- Down-sampled contact maps at 10%, 20%, 30%, 40%, 50%, 60%, 70%, and 80% of original depth per replicate
- Significant differential compartment bin sets per down-sampling level
- Depth-vs-recall table (downsampling_recall.csv) including 3v3 recovery counts (1906/1981 bins)

## How to apply

Generate down-sampled contact maps at 10%-80% of original depth for every replicate (ESC: 4 replicates, 230M-1.2B reads; NPC: 3 replicates, 720M-1.5B reads), then re-run dcHiC at 100 kb resolution on each down-sampled pairwise ESC-vs-NPC set using the RefineY_Rconf90_FDR10 settings: PCA of the observed/expected-normalized correlation matrix, quantile normalization of compartment scores across replicates, and Mahalanobis-distance outlier testing with chi-square significance and FDR10 correction. For each depth, extract the significant differential bins and compute recall against the full-depth differential bin set. Validate that recall exceeds 80% for depths of 40% and above, and additionally track recovery of the 1906 significant bins in the 3v3 analysis (dropping the 230M-read ESC replicate out of the 1981 full-set bins). Write the depth-vs-recall results to a table such as downsampling_recall.csv.

## Related tools

- **dcHiC** (Rerun of the compartment pipeline (PCA of observed/expected-normalized correlation matrix, quantile normalization, Mahalanobis-distance outlier testing with chi-square significance and FDR10) on each down-sampled pairwise map set) — https://github.com/ay-lab/dcHiC

## Evaluation signals

- Recall of full-depth differential bins exceeds 80% for all down-sampling depths of 40% and above
- 1906 of the 1981 full-depth significant bins are recovered in the 3v3 analysis (ESC replicate with 230M reads excluded)
- Each down-sampled replicate reproduces the same dcHiC settings (RefineY_Rconf90_FDR10) as the full-depth reference run
- downsampling_recall.csv contains a monotone depth-vs-recall table across the 10%-80% levels
- Down-sampled run bin counts converge toward the full-depth count as depth increases

## Limitations

- Compartment identification is constrained by map sparsity; the paper notes genomic compartment identification remains a major issue due to sparse single-cell maps even at coarse resolution
- Recall is measured only against the tool's own full-depth calls, so it gauges robustness to depth, not biological correctness
- Depth-recall relationships are dataset-specific (ESC/NPC read counts span 230M-1.5B reads) and may not transfer to other cell types or resolutions

## Evidence

- [intro] Quantile normalization and Mahalanobis pipeline: "We follow this by quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates at a time"
- [intro] Significance testing step: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [readme] Tool identity: "dcHiC is a tool for differential compartment analysis of Hi-C datasets."
- [intro] Resolution capability: "compartment analysis can be conducted on Hi-C maps with or without replicates at resolutions up to 10 kb"
