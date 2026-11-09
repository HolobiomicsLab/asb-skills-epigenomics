---
name: chromatin-loop-calling-from-imaging-data
description: Use when the input is per-cell, per-allele 3D coordinates of genomic
  loci from multiplexed DNA FISH or DNA seqFISH+ (e.g., 25 Kb or 5 Kb bins) and the
  goal is de novo chromatin loop detection and benchmarking against HiCCUPS references.
when_to_use_negative:
- Input is proximity-ligation assay data (Hi-C / Micro-C contact count matrices) —
  existing count-based loop callers such as HiCCUPS are the appropriate choice, not
  SnapFISH
- Only pairwise distances or heatmaps are needed without loop calling — use 'SnapFISH
  step1' instead of the full call-loops pipeline
- Multi-way (3+ locus) chromatin interactions are the target — SnapFISH only considers
  pairwise interactions in this work
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: SnapFISH
  role: Command-line pipeline that computes Euclidean distances, performs local-background
    T-tests/Wilcoxon tests, and calls loop candidates and summits
  repo: https://github.com/HuMingLab/SnapFISH
- name: HiCCUPS
  role: Count-based loop caller on bulk Hi-C data used as the reference loop set for
    precision/recall benchmarking of SnapFISH calls
- name: Python 3.6.8
  role: Runtime environment for SnapFISH (with numpy 1.19.5, pandas 1.1.5, scipy 1.5.4,
    statsmodels 0.12.2, matplotlib 3.3.4, click 7.1.2)
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:6ff995d3c918d8705bfbcbd6e78f4b088d2d84957528c1ac8ae27a321fa3025b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapFISH
  tools:
  - HiCCUPS
  - Python
  - Python 3.6.8
  - SnapFISH
  - matplotlib
  - numpy
  - pandas
  - scipy
  - statsmodels
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# chromatin-loop-calling-from-imaging-data

## Summary

Identify de novo chromatin loops from multiplexed DNA FISH / chromatin tracing data using SnapFISH, which models continuous 3D Euclidean distances with a local-background two-sample T-test rather than count-based proximity-ligation statistics.

## When to use

Use when the input is per-cell, per-allele 3D coordinates of genomic loci from multiplexed DNA FISH or DNA seqFISH+ (e.g., 25 Kb or 5 Kb bins) and the goal is de novo chromatin loop detection and benchmarking against HiCCUPS references.

## When NOT to use

- Input is proximity-ligation assay data (Hi-C / Micro-C contact count matrices) — existing count-based loop callers such as HiCCUPS are the appropriate choice, not SnapFISH
- Only pairwise distances or heatmaps are needed without loop calling — use 'SnapFISH step1' instead of the full call-loops pipeline
- Multi-way (3+ locus) chromatin interactions are the target — SnapFISH only considers pairwise interactions in this work

## Inputs

- 3D_COOR.txt: tab-separated per-cell, per-allele 3D coordinates with columns chr, cell_id, pos, x, y, z (unit: nm)
- ANN.txt: 1D genomic annotation file with columns chr, start, end, pos for each bin
- Optional reference loop anchors (e.g., HiCCUPS loops from deeply sequenced mESC bulk Hi-C at 10 Kb or 25 Kb) for benchmarking

## Outputs

- output_3D_dist_<suffix>.txt: pairwise Euclidean distance between each bin in each cell (missing values skipped)
- output_3D_dist_<suffix>_avg.txt: average pairwise distance across all cells
- output_test_<suffix>.txt: T-test and Wilcoxon rank test results from the local background model
- output_loop_candidate_<suffix>.txt: loop candidates passing FDR < 0.1 (T < 0)
- output_loop_summit_<suffix>.txt: final loop summits after clustering and contact-frequency filtering
- Heatmaps (PDF) of average Euclidean distance, contact frequency, loop candidates, and loop summits when -p 1

## How to apply

Format the 3D coordinates as a tab-separated 3D_COOR.txt (chr, cell_id, pos, x, y, z in nm) plus a 1D ANN.txt (chr, start, end, pos), then run 'SnapFISH call-loops -i ... -o OUT/DIR -a ... -p 1 -d SUFFIX'. SnapFISH computes pairwise Euclidean distances per cell and average distance matrices (Step 1), performs two-sample T-tests against a local background defined as identified pairs within a 25–50 Kb square of 1D genomic distance (Step 2), and calls loop candidates requiring T < 0 and FDR < 0.1 (default -h threshold). Nearby candidates within a pre-specified gap are grouped into clusters and only summits with minimal FDR are kept; the final loop list retains cluster summits with population-level contact frequency >= 1/3 and singletons >= 1/2. If distance distributions deviate from normality, re-filter with 'SnapFISH test ... -m w -h 0.1' using the Wilcoxon rank-test FDR. Validate by confirming a strongly negative Pearson correlation between the average distance and contact frequency matrices (e.g., r = -0.9845 in the Sox2 demo) before trusting loop calls.

## Related tools

- **SnapFISH** (Command-line pipeline that computes Euclidean distances, performs local-background T-tests/Wilcoxon tests, and calls loop candidates and summits) — https://github.com/HuMingLab/SnapFISH
- **HiCCUPS** (Count-based loop caller on bulk Hi-C data used as the reference loop set for precision/recall benchmarking of SnapFISH calls)
- **Python 3.6.8** (Runtime environment for SnapFISH (with numpy 1.19.5, pandas 1.1.5, scipy 1.5.4, statsmodels 0.12.2, matplotlib 3.3.4, click 7.1.2))

## Examples

```
SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129
```

## Evaluation signals

- Pearson correlation between the average Euclidean distance matrix and the contact frequency matrix must be strongly negative (demo value r = -0.9845 for 129chr3)
- Loop candidates require T < 0 and FDR < 10% (default -h 0.1); verify output_test file filtering matches this
- Benchmark on mESC DNA seqFISH+ (446 cells, 892 alleles) should reproduce ~16 loops with precision = 87.5%, recall = 40.0%, F1 = 0.549 vs HiCCUPS
- Final loop list must satisfy frequency filters: cluster summits with contact frequency >= 1/3, singletons >= 1/2
- For the Sox2 demo, exactly one summit should fall between the Sox2 promoter and super-enhancer anchors within 10 Kb in both CAST and 129 alleles

## Limitations

- Sensitivity can be improved, particularly on DNA seqFISH+ data in mESCs (recall only 40.0% against HiCCUPS on the seqFISH+ benchmark)
- Missing 3D coordinates/distances reduce sample size; imputation proposed but not implemented
- Only pairwise chromatin interactions are considered; multi-way interactions are not detected
- Default T-test assumes approximately normal distance distributions; use -m w (Wilcoxon) when distributions deviate from normality

## Evidence

- [discussion] we applied two sample T-test, instead of paired T-test, to evaluate the statistical significance of the difference in Euclidean distance: "we applied two sample T-test, instead of paired T-test, to evaluate the statistical significance of the difference in Euclidean distance"
- [discussion] The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2: "The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2"
- [methods] Take 3D coordinates of each locus in each cell as input, calculate Eucildean disance between any two loci pairs: "Step 1a: Take 3D coordinates of each locus in each cell as input, calculate Eucildean disance between any two loci pairs"
- [methods] SnapFISH is a published package on PyPI and can be downloaded by `pip install SnapFISH`: "SnapFISH is a published package on [PyPI](https://pypi.org/project/SnapFISH/0.1.0/) and can be downloaded by `pip install SnapFISH`"
- [readme] SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129: "SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129"
- [intro] All existing loop callers are designed for genomic data generated from proximity-ligation assays, which utilize the count-based statistical framework: [evidence span withheld — anchor status: not_found]
