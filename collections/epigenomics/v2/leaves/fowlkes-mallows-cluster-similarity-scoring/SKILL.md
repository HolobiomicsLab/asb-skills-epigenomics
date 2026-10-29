---
name: fowlkes-mallows-cluster-similarity-scoring
description: Use when comparing two partitions of the same genomic bins (e.g., predicted
  TAD calls vs. ground-truth TADs from simulated Hi-C datasets such as the Wang et
  al. IMR-90 noise series) to produce a similarity score per method per noise level.
when_to_use_negative:
- When only a single clustering exists and no ground-truth partition is available
  for comparison — FMS is undefined without a reference partition
- When the goal is physical/structural TAD quality (intra(i) - inter(i,j) contact
  enrichment) rather than partition agreement — use the TAD_quality metric instead
- When comparing partitions of different genomic bin sizes or different bin counts,
  since bin-level TP/FP/FN counting requires a shared binning of the chromosome
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: TOAST
  role: Produces the candidate TAD calls whose boundary bins are scored against ground
    truth by the FMS evaluation scripts
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Unsupervised clustering that assigns labels to each genomic bin; contiguous
    same-label regions define the TAD boundaries being compared via FMS
- name: TopDom
  role: Baseline TAD caller run on the same simulated matrices so its FMS can be compared
    against TOAST's
- name: HiCSeg
  role: Baseline statistical TAD caller (maximum likelihood with one-dimensional segmentation)
    included in the per-noise-level FMS comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:b575425f08e4d9aa827268ffa6e6eaa25a5abce26fce53a7dead3267f551e872
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Fowlkes-Mallows cluster similarity scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the Fowlkes-Mallows score (FMS = sqrt(TP/(TP+FP) * TP/(TP+FN))) to quantify agreement between algorithmically called TADs and ground-truth TADs in Hi-C data, used to benchmark TOAST against TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, and CASPIAN on simulated contact matrices.

## When to use

Use when comparing two partitions of the same genomic bins (e.g., predicted TAD calls vs. ground-truth TADs from simulated Hi-C datasets such as the Wang et al. IMR-90 noise series) to produce a similarity score per method per noise level.

## When NOT to use

- When only a single clustering exists and no ground-truth partition is available for comparison — FMS is undefined without a reference partition
- When the goal is physical/structural TAD quality (intra(i) - inter(i,j) contact enrichment) rather than partition agreement — use the TAD_quality metric instead
- When comparing partitions of different genomic bin sizes or different bin counts, since bin-level TP/FP/FN counting requires a shared binning of the chromosome

## Inputs

- Called TAD boundary file per method (e.g., TOAST merge-file output from toast.py)
- Ground-truth TAD boundary definitions from simulated Hi-C datasets (Wang et al. IMR-90 noise levels 4, 8, 12, 16, 20)
- NxN intra-chromosomal Hi-C contact matrix defining genomic bin size (e.g., 50 Kb bins)

## Outputs

- Per-method, per-noise-level FMS values (e.g., simulated_benchmark_table.csv)
- TOAST FMS values expected to match Tables 1-2 of the paper (~0.99 at noise 4-8, ~0.975-0.981 at noise 12-20)

## How to apply

For each genomic bin, classify it as boundary/within/between based on the called TADs and on the ground-truth TADs, then derive true positives, false positives, and false negatives over the bin-partition pairings. Compute FMS as sqrt(TP/(TP+FP) * TP/(TP+FN)), yielding 0 for unrelated partitions and 1 for identical partitions. In the TOAST workflow, called TADs come from contiguous same-label HDBSCAN regions longer than 200 Kb, and ground truth comes from the simulated dataset's known TAD boundaries. Repeat per noise level (4, 8, 12, 16, 20 on the Wang et al. IMR-90 simulated matrices) and per method, then assemble values into a per-method, per-noise-level benchmark table (e.g., simulated_benchmark_table.csv). Rationale: FMS captures both precision and recall of boundary bin agreement, enabling a single comparable number across callers like TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, and CASPIAN.

## Related tools

- **TOAST** (Produces the candidate TAD calls whose boundary bins are scored against ground truth by the FMS evaluation scripts) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Unsupervised clustering that assigns labels to each genomic bin; contiguous same-label regions define the TAD boundaries being compared via FMS)
- **TopDom** (Baseline TAD caller run on the same simulated matrices so its FMS can be compared against TOAST's)
- **HiCSeg** (Baseline statistical TAD caller (maximum likelihood with one-dimensional segmentation) included in the per-noise-level FMS comparison)

## Examples

```
python src/toast.py  # after TAD calling: python src/evaulate_TAD.py to compute FMS vs. ground truth per noise level
```

## Evaluation signals

- TOAST's overall FMS on the Wang et al. simulated data should remain around 0.98, with per-noise-level values matching Tables 1-2 (~0.99 at noise 4-8, decreasing to ~0.975-0.981 at noise 12-20)
- FMS must be 0 for two completely unrelated partitions and 1 for identical partitions (sanity check of the sqrt(TP/(TP+FP) * TP/(TP+FN)) computation)
- Every genomic bin must be assigned exactly one of boundary/within/between in both the called and ground-truth partitions before counting TP/FP/FN
- The number of TADs called by TOAST should differ from ground truth by only 1-8 TADs, and average TAD length by up to 5 Kb, on the simulated data
- Per-method FMS values for TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, and CASPIAN should be computable on the same five noise levels using the caller scripts in the TOAST repository

## Limitations

- FMS requires a defined ground-truth partition, so it is applicable to simulated datasets but cannot be computed on real Hi-C data (e.g., GM12878) without external annotations
- FMS alone does not capture contact-matrix structural quality of TADs; the paper complements it with a TAD_quality score (intra(i) - inter(i,j) averaged over called TADs) and with TAD-count/average-length differences
- The repository's evaluation script filenames contain typos ('evaulate_TAD.py'), and README documentation of folder contents is sparse, requiring inspection of /src scripts

## Evidence
