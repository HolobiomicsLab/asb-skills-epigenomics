---
name: allele-stratification-by-detection-efficiency
description: Use when evaluating how per-allele data quality (fraction of detected
  targeted segments in chromatin tracing/FISH data) affects chromatin loop caller
  sensitivity or false-positive discovery, or when deciding a detection-efficiency
  cutoff before loop calling.
when_to_use_negative:
- Input is bulk Hi-C / proximity-ligation data with count-based contacts — SnapFISH's
  Euclidean-distance local background T-test framework is designed for imaging-based
  tracing data, not count matrices.
- The goal is to detect a novel loop de novo in a full dataset, not to benchmark caller
  sensitivity — stratifying by detection efficiency reduces sample size and is a quality-control/benchmarking
  manipulation.
- Per-allele detection efficiency metadata is unavailable and cannot be computed,
  since the stratification itself depends on the fraction of detected targeted segments
  per allele.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: SnapFISH
  role: Calls chromatin loops from each detection-efficiency-stratified subset via
    `SnapFISH call-loops` on the 3D coordinates and annotation files
  repo: https://github.com/HuMingLab/SnapFISH
- name: pandas
  role: Splits the 3D coordinate file into six allele/detection-efficiency subsets
- name: scipy
  role: Two-sample T-test underlying the local background model used to identify loop
    candidates
- name: statsmodels
  role: FDR computation for filtering loop candidates at FDR < 0.1
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:8ee98bc8848320c79b46f5596cade613fcc660b0fa186d982a70d8f74b3587bd
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# allele-stratification-by-detection-efficiency

## Summary

Stratifies chromatin tracing alleles by targeted segment detection efficiency (fraction of loci imaged per allele) before running SnapFISH loop calling, to quantify how data quality controls sensitivity and false-positive rates. Applied to the mESC Sox2 locus tracing data (Huang et al, NG, 2021) split into high (~85.4%/82.6%), median (~73.2%/70.4%), and low (~57.0%/55.0%) detection efficiency groups per allele (CAST/129).

## When to use

Use when evaluating how per-allele data quality (fraction of detected targeted segments in chromatin tracing/FISH data) affects chromatin loop caller sensitivity or false-positive discovery, or when deciding a detection-efficiency cutoff before loop calling.

## When NOT to use

- Input is bulk Hi-C / proximity-ligation data with count-based contacts — SnapFISH's Euclidean-distance local background T-test framework is designed for imaging-based tracing data, not count matrices.
- The goal is to detect a novel loop de novo in a full dataset, not to benchmark caller sensitivity — stratifying by detection efficiency reduces sample size and is a quality-control/benchmarking manipulation.
- Per-allele detection efficiency metadata is unavailable and cannot be computed, since the stratification itself depends on the fraction of detected targeted segments per allele.

## Inputs

- 3D coordinate file (tab-separated: chr, cell_id, pos, x, y, z; coordinates in nm) from chromatin tracing of the mESC Sox2 locus (mm10 chr3:34,601,078-34,806,078, 41 loci at 5Kb resolution)
- 1D genomic annotation file (ANN.txt: chr, start, end, pos) for the 5Kb bins
- Per-allele detection efficiency metadata used to assign each allele to high/median/low groups

## Outputs

- Six stratified 3D coordinate subsets (CAST/129 x high/median/low detection efficiency)
- Per-subset SnapFISH loop call sets (loop candidate and loop summit files, e.g. output_loop_summit_*.txt)
- Comparison table of called loops across detection efficiency groups flagging false positives (3 false positives in low-efficiency CAST group; none in 129 groups)

## How to apply

Using pandas, split the 3D coordinate file (chr, cell_id, pos, x, y, z columns) into six subsets by allele (CAST, 129) and detection efficiency group (high ~85.4%/82.6%, median ~73.2%/70.4%, low ~57.0%/55.0%), keeping 1416 alleles per allele type for balanced comparison. For each subset, run `SnapFISH call-loops` on the 3D coordinates and 1D annotation file to compute pairwise Euclidean distances, the average distance matrix, and the local background model (segment pairs within 25-50 Kb 1D genomic distance) with a two-sample T-test (scipy/statsmodels). Filter candidates at FDR < 0.1 (SnapFISH `-h 0.1` default) with T < 0, group nearby candidates into clusters, retain summits, and apply the population-level contact frequency thresholds (cluster summits >= 1/3, singletons >= 1/2). Compare called loops across the six groups against the Sox2 enhancer-promoter loop to identify false positives induced by low detection efficiency. The rationale is that lower segment detection reduces effective sample size and statistical power, so low-efficiency groups reveal the reliability floor of the caller.

## Related tools

- **SnapFISH** (Calls chromatin loops from each detection-efficiency-stratified subset via `SnapFISH call-loops` on the 3D coordinates and annotation files) — https://github.com/HuMingLab/SnapFISH
- **pandas** (Splits the 3D coordinate file into six allele/detection-efficiency subsets)
- **scipy** (Two-sample T-test underlying the local background model used to identify loop candidates)
- **statsmodels** (FDR computation for filtering loop candidates at FDR < 0.1)

## Examples

```
SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129
```

## Evaluation signals

- The low-efficiency CAST group contains exactly 3 false-positive loop calls relative to the known Sox2 enhancer-promoter loop, while no false positives appear in any 129 group.
- SnapFISH's reported cutoff equals the average Euclidean distance in nm between bin pairs 25Kb apart (e.g. 349.4659 for the 129 demo), and Pearson's r between average distance and contact frequency matrices is strongly negative (~-0.98).
- Each retained loop satisfies T < 0 and FDR < 0.1, and final loops meet population-level contact frequency >= 1/3 (cluster summits) or >= 1/2 (singletons).
- Each allele group contains the expected number of alleles (1416 per allele type) and detection efficiency near the group target (high ~85%/83%, median ~73%/70%, low ~57%/55%).
- Loops overlapping Sox2 enhancer-promoter anchors within 10 Kb are recovered in the high and median detection efficiency groups.

## Limitations

- Sensitivity of SnapFISH degrades at low detection efficiency — the low-efficiency CAST group yielded 3 false positives, indicating calls from sparse alleles are less reliable.
- SnapFISH considers only pair-wise chromatin interactions; multi-way interactions are not detected.
- Imputation of missing 3D coordinates/distances is proposed but not yet implemented, limiting statistical power in low-detection-efficiency subsets.

## Evidence

- [discussion] Definition of local background window: "we define the local neighborhood of a given targeted segment pair as all of the identified pairs that fall within a square with 25 Kb ~ 50 Kb in 1D genomic distance"
- [discussion] Candidate filtering thresholds: "(T < 0) and FDR < 10%"
- [discussion] Final loop contact-frequency thresholds: "cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2"
- [readme] FDR threshold parameter default: "-h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1."
- [readme] Dataset description for stratified alleles: "This dataset contains two alleles: CAST and 129. The CAST allele contains a 7.5Kb 4CBS insertion"
