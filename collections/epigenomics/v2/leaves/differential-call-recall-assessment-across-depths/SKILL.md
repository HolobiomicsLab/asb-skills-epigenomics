---
name: differential-call-recall-assessment-across-depths
description: Use when you need to test whether pairwise differential compartment calls
  remain stable at reduced sequencing depth, e.g., before committing to lower-depth
  runs or justifying shallow replicate removal.
when_to_use_negative:
- Your goal is calling differential compartments once at full depth rather than assessing
  their robustness to sequencing depth
- Data are single-cell Hi-C maps so sparse that down-sampling to 10-80% would yield
  uninformative compartments; dcHiC's own caveat is that compartment identification
  from single-cell maps 'is still a major issue in this field due to sparsity'
- Your comparison involves more than two conditions or a time-series design where
  pairwise recall against a single reference set is not the appropriate robustness
  metric
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0607
- http://edamontology.org/topic_3168
tools:
- name: dcHiC
  role: Runs the compartment pipeline (PCA of observed/expected-normalized correlation
    matrix, quantile normalization, Mahalanobis-distance chi-square testing with FDR
    correction) on each down-sampled and full-depth pairwise comparison
  repo: https://github.com/ay-lab/dcHiC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34626-6
    title: dcHiC detects differential compartments across multiple Hi-C datasets
  source_content_hash: sha256:d95623d0bcef8d29219b7d313f9518831462989cd0b459c3fc1f4e6859a5bf9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ay-lab/dcHiC
  tools:
  - dcHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0607
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# differential-call-recall-assessment-across-depths

## Summary

A robustness assessment skill that down-samples Hi-C replicates to graded sequencing depths and measures how well dcHiC's differential compartment calls (e.g., ESC vs NPC at 100 kb) are recalled relative to full-depth results.

## When to use

Use when you need to test whether pairwise differential compartment calls remain stable at reduced sequencing depth, e.g., before committing to lower-depth runs or justifying shallow replicate removal.

## When NOT to use

- Your goal is calling differential compartments once at full depth rather than assessing their robustness to sequencing depth
- Data are single-cell Hi-C maps so sparse that down-sampling to 10-80% would yield uninformative compartments; dcHiC's own caveat is that compartment identification from single-cell maps 'is still a major issue in this field due to sparsity'
- Your comparison involves more than two conditions or a time-series design where pairwise recall against a single reference set is not the appropriate robustness metric

## Inputs

- Full-depth Hi-C contact maps per replicate for two conditions (ESC: 4 replicates, 230M-1.2B reads; NPC: 3 replicates, 720M-1.5B reads)
- Down-sampled contact maps at 10%, 20%, 30%, 40%, 50%, 60%, 70%, and 80% of original sequencing depth for every replicate
- Full-depth significant differential compartment bin set at 100 kb resolution (reference, 1981 bins)

## Outputs

- Per-depth significant differential compartment bin sets at 100 kb resolution
- Depth-vs-recall table (downsampling_recall.csv) including recall per down-sampling level and 3v3 recovery counts
- Recall threshold verdict (e.g., >80% recall at >=40% depth; 1906 of 1981 bins recovered in 3v3)

## How to apply

Load full-depth contact maps for both conditions (here 4 ESC replicates at 230M–1.2B reads and 3 NPC replicates at 720M–1.5B reads) and generate down-sampled maps at 10%–80% of original depth per replicate. Rerun dcHiC's pipeline on each down-sampled pairwise set at 100 kb resolution: observed/expected normalization and PCA of the correlation matrix, quantile normalization of compartment scores across replicates, and Mahalanobis-distance outlier testing with chi-square significance and FDR correction (FDR10, e.g., RefineY_Rconf90_FDR10). Extract significant differential bins from each run and compute recall against the full-depth differential bin set per down-sampling level. Track additionally the fraction of the 1906 significant bins recovered when excluding the low-depth (230M read) ESC replicate in a 3v3 comparison. Write a depth-vs-recall table (downsampling_recall.csv) and consider depth sufficient when recall exceeds 80%.

## Related tools

- **dcHiC** (Runs the compartment pipeline (PCA of observed/expected-normalized correlation matrix, quantile normalization, Mahalanobis-distance chi-square testing with FDR correction) on each down-sampled and full-depth pairwise comparison) — https://github.com/ay-lab/dcHiC

## Examples

```
Rscript dchic_run.R --input esc_full,npc_full --resolution 100kb --fdr 0.10 --filter RefineY_Rconf90_FDR10; Rscript compute_recall.R --ref full_depth_bins --downsampled 10:10:80 --out downsampling_recall.csv
```

## Evaluation signals

- Recall exceeds 80% for down-sampling depths of 40% and above at 100 kb resolution
- The 3v3 analysis (excluding the 230M-read ESC replicate) recovers 1906 of the 1981 significant bins
- Each run uses the same filter configuration as the reference (FDR10 setting, e.g., RefineY_Rconf90_FDR10) so calls are comparable across depths
- downsampling_recall.csv contains one recall value per down-sampling level (10% through 80%) computed against the same full-depth bin set
- Down-sampled replicates match exactly the stated fractions of original read counts (10%-80%)

## Limitations

- Single-cell Hi-C sparsity limits compartment identification even at coarse resolution, so extreme down-sampling approaches this regime
- Recall is measured only against dcHiC's own full-depth calls at one resolution (100 kb) and one filter setting (FDR10); conclusions may not transfer to other resolutions or significance thresholds
- An outlier low-depth replicate (ESC, 230M reads) depresses recovery, motivating the 3v3 exclusion analysis

## Evidence

- [intro] dcHiC uses quantile normalization to obtain comparable compartment scores across two or more Hi-C maps/replicates and a multivariate distance measure to estimate statistical significance of compartment differences: [evidence span withheld — anchor status: not_found]
- [intro] for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier: "for each genomic bin, dcHiC calculates the Mahalanobis distance, a multi-variate z-score that measures the extent to which each bin is an outlier"
- [discussion] Genomic compartment identification from single-cell Hi-C maps is still a major issue in this field due to sparsity of single-cell contact maps even at coarse resolution.: "Genomic compartment identification from single-cell Hi-C maps is still a major issue in this field due to sparsity of single-cell contact maps"
- [methods] RefineY_Rconf90_FDR10.pcOri.html: "RefineY_Rconf90_FDR10.pcOri.html"
- [readme] Optimized PCA calculations (faster + capable of analysis up to 5kb resolution): "Optimized PCA calculations (faster + capable of analysis up to 5kb resolution)"
