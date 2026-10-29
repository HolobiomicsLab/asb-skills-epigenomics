---
name: simulated-hi-c-benchmark-comparison
description: Use when evaluating or comparing TAD identification algorithms on simulated
  Hi-C contact matrices (e.g., Wang et al. IMR-90 noise levels 4-20) where ground-truth
  TAD boundaries are known and FMS/TAD-quality metrics are required.
when_to_use_negative:
- The evaluation target is real Hi-C data without ground-truth TADs — FMS against
  truth is undefined; use enrichment/anchoring analyses instead.
- The task is identifying TADs on a single new Hi-C matrix rather than benchmarking
  caller performance across noise levels.
- Input data are polymer-model simulated genomes from Trussart et al. or Forcato et
  al. toy genomes without Wang et al. noise-level ground truth — metric interpretation
  differs.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: TOAST
  role: 'TAD caller under test: splits matrices into 400x400 submatrices, trains a
    2-layer GAE for 16-dim embeddings, clusters with HDBSCAN, and calls TADs from
    contiguous labels'
  repo: https://github.com/ghaiyan/TOAST
- name: HDBSCAN
  role: Unsupervised clustering of GAE embeddings to assign labels to each genomic
    bin
- name: TopDom
  role: Baseline TAD-calling method run on the same five simulated matrices for FMS/TAD-quality
    comparison
- name: HiCSeg
  role: Baseline statistical TAD caller (maximum likelihood + one-dimensional segmentation)
    included in the comparison
- name: CASPIAN
  role: Baseline caller taking a tab-separated N by N intrachromosomal contact matrix
    as input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:f65f6d501293bdeecb6165263cc18b7b177d4b3948f81b4b0d51bdb85ad09e40
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
  - CASPIAN
  - HDBSCAN
  - HiCSeg
  - TOAST
  - TopDom
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# simulated-hi-c-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a TAD-calling method (e.g., TOAST) against baselines such as TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, and CASPIAN on simulated Hi-C contact matrices across noise levels 4-20, scoring each caller with the Fowlkes-Mallows score (FMS) and a TAD quality score against ground-truth TADs.

## When to use

Use when evaluating or comparing TAD identification algorithms on simulated Hi-C contact matrices (e.g., Wang et al. IMR-90 noise levels 4-20) where ground-truth TAD boundaries are known and FMS/TAD-quality metrics are required.

## When NOT to use

- The evaluation target is real Hi-C data without ground-truth TADs — FMS against truth is undefined; use enrichment/anchoring analyses instead.
- The task is identifying TADs on a single new Hi-C matrix rather than benchmarking caller performance across noise levels.
- Input data are polymer-model simulated genomes from Trussart et al. or Forcato et al. toy genomes without Wang et al. noise-level ground truth — metric interpretation differs.

## Inputs

- Wang et al. simulated Hi-C contact matrices (IMR-90 cell line) at noise levels 4, 8, 12, 16, 20 in NxN intrachromosomal matrix format
- Ground-truth TAD boundary annotations for each simulated matrix
- Caller scripts for TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, and CASPIAN from https://github.com/ghaiyan/TOAST

## Outputs

- Per-method, per-noise-level FMS values (simulated_benchmark_table.csv)
- Per-method, per-noise-level TAD_quality scores (intra minus inter contacts averaged over called TADs)
- Called TAD boundary files per method per noise level

## How to apply

Load the Wang et al. simulated IMR-90 contact matrices at noise levels 4, 8, 12, 16, and 20 as NxN intrachromosomal matrices, the NxN format the TOAST pipeline uses as input. Run TOAST by splitting each matrix into non-overlapping 400x400 submatrices, training a 2-layer GAE (hidden_dim=64, embedding_dim=16, lr=0.001, 200 epochs, Adam, MSE loss) to obtain 16-dimensional embeddings, clustering with HDBSCAN, and keeping contiguous same-label regions longer than 200 Kb as TADs. Run the baselines (TopDom, HiCSeg, IC-Finder, Insulation Score, ClusterTAD, CASPIAN) on the same five matrices using the caller scripts from the TOAST repository. For each method and noise level, compute FMS = sqrt(TP/(TP+FP) * TP/(TP+FN)) against ground truth by binning genomic bins into boundary/within/between clusters, and compute TAD_quality as intra(i) - inter(i,j) averaged over called TADs. Assemble per-method, per-noise-level FMS and TAD_quality values into a table; validate that TOAST FMS values match the reported ranges (~0.99 at noise 4-8, ~0.975-0.981 at noise 12-20, overall around 0.98).

## Related tools

- **TOAST** (TAD caller under test: splits matrices into 400x400 submatrices, trains a 2-layer GAE for 16-dim embeddings, clusters with HDBSCAN, and calls TADs from contiguous labels) — https://github.com/ghaiyan/TOAST
- **HDBSCAN** (Unsupervised clustering of GAE embeddings to assign labels to each genomic bin)
- **TopDom** (Baseline TAD-calling method run on the same five simulated matrices for FMS/TAD-quality comparison)
- **HiCSeg** (Baseline statistical TAD caller (maximum likelihood + one-dimensional segmentation) included in the comparison)
- **CASPIAN** (Baseline caller taking a tab-separated N by N intrachromosomal contact matrix as input)

## Examples

```
python src/toast.py --input wang_imr90_noise16_matrix.txt --output tads_noise16/ && python src/evaulate_TAD.py --pred tads_noise16/TAD.bed --truth wang_imr90_noise16_truth.bed
```

## Evaluation signals

- TOAST FMS at noise levels 4-8 should be approximately 0.99, decreasing to about 0.975-0.981 at noise 12-20, with overall FMS around 0.98, matching Tables 1-2.
- At noise level 16, TOAST and TopDom should achieve the highest FMS values among all methods.
- TOAST, TopDom, and HiCSeg should yield TAD quality scores surpassing 10 at noise level 16.
- Difference in TAD number vs. ground truth should be within 1-8 TADs and average TAD length difference up to about 5 Kb.
- All methods must be run on identical NxN simulated matrices per noise level so FMS/TAD_quality differences reflect caller behavior, not input differences.

## Limitations

- FMS validation depends on the Wang et al. ground-truth annotations; the skill does not apply where truth labels are unavailable.
- TOAST's GAE requires dividing contact matrices into non-overlapping 400x400 submatrices, which constrains usable matrix sizes and resolutions.
- Performance degrades with increasing noise: FMS decreases from ~0.99 at noise 4-8 to ~0.975-0.981 at noise 12-20.

## Evidence
