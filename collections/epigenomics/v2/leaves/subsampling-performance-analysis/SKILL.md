---
name: subsampling-performance-analysis
description: Use when evaluating whether a computational method's detection performance
  (precision/recall/F1) degrades gracefully with reduced input depth, e.g. subsampling
  single-cell Hi-C contact files and comparing calls to a bulk-derived reference loop
  set.
when_to_use_negative:
- The goal is de novo loop detection rather than benchmarking an existing method against
  a reference loop list
- Only a single full-depth dataset is available with no subsampling series and no
  combined reference loop set to compute TP/FP/FN against
- The candidate method cannot emit loop coordinates in a bedpe-like anchor format
  that supports genomic interval overlap matching
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0622
tools:
- name: SnapHiC
  role: Method under test; identifies chromatin loops from scHi-C data and emits *.postprocessed.summits.bedpe
    loop summits at each subsampling level
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Merges reference loop sources, performs anchor-overlap matching, counts TP/FP/FN,
    and computes precision/recall/F1
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:73c89c221e79c29355ef4fe80b1bc77279e388a7e872bb8fa5ad775475e2dc07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# subsampling-performance-analysis

## Summary

Benchmark a loop-calling method's performance across successive subsamplings of single-cell Hi-C data by computing precision, recall, and F1 against a combined reference loop list, as done when comparing SnapHiC to HiCCUPS, FastHiC, FitHiC2 and HiC-ACT on 742 mouse embryonic stem cell scHi-C datasets.

## When to use

Use when evaluating whether a computational method's detection performance (precision/recall/F1) degrades gracefully with reduced input depth, e.g. subsampling single-cell Hi-C contact files and comparing calls to a bulk-derived reference loop set.

## When NOT to use

- The goal is de novo loop detection rather than benchmarking an existing method against a reference loop list
- Only a single full-depth dataset is available with no subsampling series and no combined reference loop set to compute TP/FP/FN against
- The candidate method cannot emit loop coordinates in a bedpe-like anchor format that supports genomic interval overlap matching

## Inputs

- Subsampled single-cell Hi-C contact files (tab-separated/gzipped per-cell contact files, one line per contact with chromosome and mapped position columns)
- Called loop sets as bedpe summit coordinates per method (e.g. SnapHiC *.postprocessed.summits.bedpe)
- Reference loop sources: HiCCUPS loops from bulk in situ Hi-C and MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP

## Outputs

- loop_calling_benchmark_f1.csv with per-method, per-subsampling precision/recall/F1 values
- Method ranking by F1 score at each subsampling level

## How to apply

Subsample the single-cell contact data (e.g. from 742 mouse embryonic stem cells) to multiple depth levels and run the method under test at each level, collecting its called loop set as bedpe summit coordinates (for SnapHiC, the *.postprocessed.summits.bedpe output). Build one combined reference loop list by merging HiCCUPS loops from bulk in situ Hi-C with MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP using Python. Match each method's called loops to the combined reference by genomic interval overlap of loop anchor coordinates, then count true positives, false positives, and false negatives per method per subsampling level. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 as the harmonic mean of precision and recall, writing per-method, per-subsampling values to loop_calling_benchmark_f1.csv and ranking methods by F1. The key decision point is the overlap-based matching of anchor coordinates, which defines TP/FP/FN counts; validation requires the method under test to attain the maximum F1 at every subsampling level.

## Related tools

- **SnapHiC** (Method under test; identifies chromatin loops from scHi-C data and emits *.postprocessed.summits.bedpe loop summits at each subsampling level) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Merges reference loop sources, performs anchor-overlap matching, counts TP/FP/FN, and computes precision/recall/F1)

## Examples

```
python benchmark_f1.py --methods snaphic.fastqc,hiccups,fasthic,fithic2,hicact --reference combined_reference_loops.bedpe --out loop_calling_benchmark_f1.csv
```

## Evaluation signals

- At every subsampling level, the tabulated F1 score of SnapHiC is the maximum among all compared methods (HiCCUPS, FastHiC, FitHiC2, HiC-ACT)
- Precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 equals the harmonic mean of precision and recall in every row of loop_calling_benchmark_f1.csv
- TP + FN for each method is constant across methods at a given subsampling level (it equals the size of the combined reference loop list)
- Each called loop is matched against the combined reference by genomic interval overlap of loop anchor coordinates, and every reference source (HiCCUPS bulk Hi-C plus three MAPS interaction sets) is included in the merge

## Limitations

- Performance is evaluated against a combined reference loop list assembled from bulk Hi-C (HiCCUPS) and bulk-derived MAPS interactions, so calls absent from those bulk assays count as false positives
- SnapHiC's benchmarking was performed on scHi-C from 742 mouse embryonic stem cells; results on other cell types or genomes (mm10/hg19/hg38 supported) may differ
- The inverse-matrix RWR computation is memory-intensive and slow on non-HPC environments; the sliding-window approximation is an alternative

## Evidence

- [results] At each subsampling of scHi-C data, SnapHiC consistently attained a greater F1 score than HiCCUPS, and also than FastHiC, FitHiC2 and HiC-ACT: [evidence span withheld — anchor status: not_found]
- [methods] Count TP/FP/FN and compute precision, recall, F1: [evidence span withheld — anchor status: not_found]
- [methods] Merge reference loop sources into combined reference list: [evidence span withheld — anchor status: not_found]
- [abstract] Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools: "Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools"
- [readme] SnapHiC postprocessed summits bedpe output: "the prefix of the output files, including *.postprocessed.summits.bedpe"
