---
name: loop-false-positive-assessment
description: Use when chromatin tracing 3D coordinate data has heterogeneous per-segment
  detection efficiency and you need to test whether a loop caller (SnapFISH) reports
  spurious loops in low-efficiency subsets.
when_to_use_negative:
- Data is proximity-ligation assay (Hi-C/micro-C) count-based contact data rather
  than continuous Euclidean distances from chromatin tracing imaging
- The goal is de novo loop discovery on a single dataset rather than assessing robustness/false
  positives across detection-efficiency strata
- The dataset has uniform, near-complete segment detection so efficiency stratification
  would produce no meaningful contrast
edam_operation: http://edamontology.org/operation_3903
edam_topics:
- http://edamontology.org/topic_3940
tools:
- name: SnapFISH
  role: Loop caller run per stratified subset via `SnapFISH call-loops` to produce
    loop candidates and summits
  repo: https://github.com/HuMingLab/SnapFISH
- name: pandas
  role: Split the 3D coordinate file into six allele x detection-efficiency subsets
- name: scipy
  role: Supplies the two-sample T-test underlying SnapFISH's local background model
- name: statsmodels
  role: FDR computation for filtering loop candidates at threshold 0.1
- name: numpy
  role: Numerical handling of pairwise Euclidean distance matrices
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:3db829f1ad36db4108910b887badad2e6c6936c47d515292617456882d4ae679
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapFISH
  tools:
  - SnapFISH
  - numpy
  - pandas
  - scipy
  - statsmodels
  edam_operation: http://edamontology.org/operation_3903
  edam_topics:
  - http://edamontology.org/topic_3940
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# loop-false-positive-assessment

## Summary

A benchmarking skill that stratifies chromatin tracing alleles by targeted segment detection efficiency and compares SnapFISH loop calls across strata to quantify how lowered detection efficiency inflates false-positive loop calls (e.g., the Sox2 enhancer-promoter loop on the weak-loop CAST allele).

## When to use

Use when chromatin tracing 3D coordinate data has heterogeneous per-segment detection efficiency and you need to test whether a loop caller (SnapFISH) reports spurious loops in low-efficiency subsets.

## When NOT to use

- Data is proximity-ligation assay (Hi-C/micro-C) count-based contact data rather than continuous Euclidean distances from chromatin tracing imaging
- The goal is de novo loop discovery on a single dataset rather than assessing robustness/false positives across detection-efficiency strata
- The dataset has uniform, near-complete segment detection so efficiency stratification would produce no meaningful contrast

## Inputs

- 3D_COOR.txt: tab-separated file with columns chr, cell_id, pos, x, y, z (3D coordinates in nm) for the mESC Sox2 locus (mm10 chr3:34,601,078-34,806,078, 41 loci at 5Kb resolution)
- ANN.txt: 1D genomic annotation file with columns chr, start, end, pos
- Per-allele targeted segment detection efficiency values used to assign high/median/low groups (CAST and 129)

## Outputs

- Six stratified loop call sets (one per allele x efficiency group) from `SnapFISH call-loops` (output_loop_summit files)
- A comparison table flagging false-positive loop calls, expected to show 3 false positives in the low-efficiency CAST group and none in any 129 group
- Confirmation of whether the known Sox2 enhancer-promoter loop is retained or lost per efficiency stratum

## How to apply

Split the 3D coordinate file with pandas into six subsets by allele (CAST, 129) and targeted segment detection efficiency group (high ~85.4%/82.6%, median ~73.2%/70.4%, low ~57.0%/55.0%), keeping 1416 alleles per allele type. Run `SnapFISH call-loops` on each subset's 3D coordinates and annotation files so the tool computes pairwise Euclidean distances per cell, the average distance matrix, and applies the local background model (segment pairs within 25-50 Kb 1D genomic distance) via two-sample T-test. Keep the default FDR filter `-h 0.1` with T < 0, cluster nearby candidates and retain summits, then apply population-level contact frequency thresholds (cluster summits >= 1/3, singletons >= 1/2). Finally, compare called loops across the six groups, flagging loops that appear only in lower-efficiency strata as detection-efficiency-induced false positives.

## Related tools

- **SnapFISH** (Loop caller run per stratified subset via `SnapFISH call-loops` to produce loop candidates and summits) — https://github.com/HuMingLab/SnapFISH
- **pandas** (Split the 3D coordinate file into six allele x detection-efficiency subsets)
- **scipy** (Supplies the two-sample T-test underlying SnapFISH's local background model)
- **statsmodels** (FDR computation for filtering loop candidates at threshold 0.1)
- **numpy** (Numerical handling of pairwise Euclidean distance matrices)

## Examples

```
SnapFISH call-loops -i ext/CAST_low_3d_coor.txt -o tmp_low_cast -a ext/input_ann.txt -p 1 -d CAST_low -h 0.1
```

## Evaluation signals

- The low-efficiency CAST group contains exactly 3 false-positive loop calls, matching the paper's report
- No false positives appear in any of the three 129 groups
- SnapFISH detects the Sox2 enhancer-promoter loop in the high and median detection efficiency CAST groups but not in the low group
- Each stratified subset retains 1416 alleles per allele type before calling
- SnapFISH run output reports a plausible cutoff (average Euclidean distance in nm between bin pairs 25 Kb apart) and a strongly negative Pearson's r between average.distance and contact.freq (demo: -0.9845)

## Limitations

- Low detection efficiency reduces sample size and statistical power; imputing missing 3D coordinates and Euclidean distances is proposed but not yet implemented in SnapFISH
- SnapFISH sensitivity can be further improved, particularly for DNA seqFISH+ data in mESCs
- Results are anchored to one locus (mESC Sox2) and one tracing dataset (Huang et al, NG, 2021); extrapolation to other loci or technologies is untested

## Evidence

- [results] For the weak-loop CAST allele, SnapFISH detected the Sox2 enhancer-promoter loop in the high and median detection efficiency groups, but reported 3 false positives in the low detection efficiency group.: [evidence span withheld — anchor status: not_found]
- [discussion] Local background model uses 25-50 Kb neighborhood with two-sample T-test: "we define the local neighborhood of a given targeted segment pair as all of the identified pairs that fall within a square with 25 Kb ~ 50 Kb in 1D genomic distance"
- [discussion] Final loop filtering thresholds for cluster summits and singletons: "The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2"
- [readme] Default FDR threshold parameter: "-h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1."
- [discussion] Loop candidates defined by T < 0 and FDR < 10%: "(T < 0) and FDR < 10%"
- [readme] Dataset description for the Sox2 locus demo: "41 loci cover a 205Kb region in mESC *Sox2* locus - chr3:34,601,078–34,806,078 (ref: mm10)"
- [discussion] Missing-data imputation as future statistical-power improvement: "We expect that imputing missing 3D coordinates and missing Euclidean distance between genomic loci of interest may increase the sample size"
