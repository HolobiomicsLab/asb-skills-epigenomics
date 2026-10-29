---
name: loop-calling-statistical-background-testing
description: Use when calling de novo chromatin loops from imaging-based chromatin
  tracing data (e.g., ORCA, multiplexed DNA FISH 3D coordinates in nm) where count-based
  proximity-ligation loop callers are not applicable.
when_to_use_negative:
- Input is Hi-C / Micro-C proximity-ligation data with contact counts — count-based
  callers like HiCCUPS or SnapHiC are designed for that framework, not Euclidean-distance
  imaging data
- The goal is multi-way (higher-order) chromatin interaction detection — SnapFISH
  only considers pairwise interactions
- Only pairwise distances are needed without loop calling — use 'SnapFISH step1' instead
  of the full test-and-filter pipeline
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: SnapFISH
  role: Command-line pipeline that computes Euclidean distances, runs the two-sample
    T-test/Wilcoxon rank test against the local background model, and calls loop candidates
    and summits
  repo: https://github.com/HuMingLab/SnapFISH
- name: scipy
  role: Statistical testing dependency (scipy 1.5.4) used for T-tests and related
    computations
- name: statsmodels
  role: Dependency (statsmodels 0.12.2) supporting FDR computation for test filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:efd6498cbededb78fbf82aee8ed8cd949996ecd8c55ebc2579a5d3cbe7ff9ca4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapFISH
  tools:
  - Python
  - SnapFISH
  - click
  - matplotlib
  - numpy
  - pandas
  - scipy
  - statsmodels
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# loop-calling-statistical-background-testing

## Summary

A statistical testing strategy from SnapFISH that identifies chromatin loops from multiplexed DNA FISH (chromatin tracing) data by comparing each locus pair's Euclidean distance against a local background model, using two-sample T-tests or Wilcoxon rank tests with FDR control.

## When to use

Use when calling de novo chromatin loops from imaging-based chromatin tracing data (e.g., ORCA, multiplexed DNA FISH 3D coordinates in nm) where count-based proximity-ligation loop callers are not applicable.

## When NOT to use

- Input is Hi-C / Micro-C proximity-ligation data with contact counts — count-based callers like HiCCUPS or SnapHiC are designed for that framework, not Euclidean-distance imaging data
- The goal is multi-way (higher-order) chromatin interaction detection — SnapFISH only considers pairwise interactions
- Only pairwise distances are needed without loop calling — use 'SnapFISH step1' instead of the full test-and-filter pipeline

## Inputs

- 3D_COOR.txt: per-cell 3D coordinates (tab-separated: chr, cell_id, pos, x, y, z in nm)
- ANN.txt: 1D genomic annotation per bin (chr, start, end, pos), e.g. 5Kb bins across mm10 chr3:34,601,078–34,806,078 Sox2 locus

## Outputs

- output_test_*.txt: T-test and Wilcoxon rank test results from the local background model
- output_loop_candidate_*.txt: loop candidate bin pairs passing T < 0 and FDR < 10%
- output_loop_summit_*.txt: loop summits after clustering, singleton removal, and contact-frequency filtering
- Heatmaps of average Euclidean distance, contact frequency, loop candidates, and loop summits (PDF)

## How to apply

Take per-cell 3D coordinates (chr, cell_id, pos, x/y/z in nm) and a 1D genomic annotation (chr, start, end, pos) as input; compute pairwise Euclidean distances and average distance and contact-frequency matrices (expect strong anti-correlation, e.g. Pearson's r ≈ -0.9845 for the mESC Sox2 129 allele demo). For each targeted segment pair, build a local background from all identified pairs within a 25–50 Kb square in 1D genomic distance and evaluate the difference in Euclidean distance with a two-sample T-test; mark loop candidates when T < 0 and FDR < 10%. If the distance distribution deviates significantly from normality, re-filter using the Wilcoxon rank test via 'SnapFISH test' with the FDR threshold flag (-h, default 0.1). Finally group nearby candidates into clusters, remove singletons, keep summits with minimal FDR, and retain cluster summits with contact frequency >=1/3 and singletons with contact frequency >=1/2.

## Related tools

- **SnapFISH** (Command-line pipeline that computes Euclidean distances, runs the two-sample T-test/Wilcoxon rank test against the local background model, and calls loop candidates and summits) — https://github.com/HuMingLab/SnapFISH
- **scipy** (Statistical testing dependency (scipy 1.5.4) used for T-tests and related computations)
- **statsmodels** (Dependency (statsmodels 0.12.2) supporting FDR computation for test filtering)

## Examples

```
SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129
```

## Evaluation signals

- Run prints the distance cutoff (average Euclidean distance between bins 25 Kb apart, e.g. 349.4659 nm for the 129chr3 demo) and Pearson's r between average distance and contact frequency matrices (expected ≈ -0.9845)
- Loop summit output file (e.g. tmp/output_loop_summit_129chr3.txt) and heatmap_loop_summits PDF exist and contain summits with FDR below the -h threshold (default 0.1)
- Called loop anchors overlap known anchors within 10 Kb — e.g. recovery of the Sox2 enhancer-promoter loop within 10 Kb of promoter-super-enhancer anchors
- Retained loops satisfy contact-frequency thresholds: cluster summits >=1/3, singletons >=1/2
- Results are consistent across imaging technologies (SnapFISH loops on chromatin tracing data match multiplexed DNA FISH identifications)

## Limitations

- All existing loop callers, including count-based frameworks, were not directly applicable to imaging data measuring continuous Euclidean distances — motivating but also constraining this approach
- Only pairwise chromatin interactions are considered; multi-way interactions are not detected
- Sensitivity can be improved, particularly for DNA seqFISH+ data in mESCs
- Missing 3D coordinates/distances are skipped rather than imputed; imputation is proposed but not implemented and could increase statistical power

## Evidence

- [discussion] two-sample T-test against local background: "we applied two sample T-test, instead of paired T-test, to evaluate the statistical significance of the difference in Euclidean distance"
- [discussion] loop candidate criteria: "(T < 0) and FDR < 10%"
- [discussion] local background definition: "we define the local neighborhood of a given targeted segment pair as all of the identified pairs that fall within a square with 25 Kb ~ 50 Kb in 1D genomic distance"
- [methods] Step 2 testing description: "Step 2: Perform two sample T-test or Wilcoxon rank test for each loci pair based on local background model"
- [methods] FDR threshold flag: "-h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1."
- [readme] expected demo output: "Pearson's r (average.dist and contact.freq): -0.9845"
