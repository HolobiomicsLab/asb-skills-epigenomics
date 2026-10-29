---
name: haplotype-specific-loop-summit-identification
description: Use when per-haplotype (e.g., CAST vs 129) chromatin loops such as the
  Sox2 promoter-super-enhancer contact must be called from allele-resolved multiplexed
  DNA FISH 3D coordinates at 5 Kb bins.
when_to_use_negative:
- Input is bulk proximity-ligation data (Hi-C contact counts) rather than continuous
  per-allele 3D Euclidean distances — count-based callers like HiCCUPS are designed
  for that data type, and existing loop callers are not directly applicable to imaging
  data
- You only need per-cell pairwise distances without loop calling — run 'SnapFISH step1'
  instead of the full call-loops workflow
- Data are not allele-resolved (no per-cell haplotype assignment), so a mixed call
  cannot yield haplotype-specific CAST vs 129 summits
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SnapFISH
  role: 'Primary pipeline: ''SnapFISH call-loops'' run once per allele with a distinct
    ''-d'' suffix computes pairwise Euclidean distances, performs the two-sample T-test
    against the 25-50 Kb local background, and groups loop candidates into clusters
    to output per-haplotype summits'
  repo: https://github.com/HuMingLab/SnapFISH
- name: HiCCUPS
  role: Orthogonal bulk Hi-C loop caller whose mESC HiCCUPS loop serves as the reference
    confirming the per-haplotype SnapFISH summit matches the known Sox2 promoter-super-enhancer
    loop
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:4ebb9ce43c5d7cc982b9a7c3b73afa9149aa21d5270a1df7bd6b912812d96e9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapFISH
  tools:
  - HiCCUPS
  - Python
  - SnapFISH
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# haplotype-specific-loop-summit-identification

## Summary

Applies the SnapFISH call-loops workflow separately to allele-resolved multiplexed DNA FISH 3D coordinates (e.g., 1,416 CAST and 1,416 129 alleles of the mESC Sox2 locus at 5 Kb bins) to call per-haplotype loop candidates and a single enhancer-promoter loop summit, checked against the known bulk Hi-C loop.

## When to use

Use when per-haplotype (e.g., CAST vs 129) chromatin loops such as the Sox2 promoter-super-enhancer contact must be called from allele-resolved multiplexed DNA FISH 3D coordinates at 5 Kb bins.

## When NOT to use

- Input is bulk proximity-ligation data (Hi-C contact counts) rather than continuous per-allele 3D Euclidean distances — count-based callers like HiCCUPS are designed for that data type, and existing loop callers are not directly applicable to imaging data
- You only need per-cell pairwise distances without loop calling — run 'SnapFISH step1' instead of the full call-loops workflow
- Data are not allele-resolved (no per-cell haplotype assignment), so a mixed call cannot yield haplotype-specific CAST vs 129 summits

## Inputs

- Per-allele 3D coordinate file (3D_COOR.txt: chr, cell_id, pos, x, y, z in nm) for each haplotype, e.g., 129_3d_coor.txt and the CAST allele coordinates
- 1D genomic annotation file (ANN.txt: chr, start, end, pos) mapping each 5 Kb bin to its location (mESC Sox2 locus, mm10 chr3:34,601,078-34,806,078)

## Outputs

- output_loop_candidate_<suffix>.txt per haplotype (44 candidates for CAST, 61 for 129 in the reported run)
- output_loop_summit_<suffix>.txt per haplotype (a single summit between the Sox2 promoter and super-enhancer in both alleles)
- output_3D_dist_<suffix>chr3.txt and output_3D_dist_<suffix>chr3_avg.txt (per-cell and average pairwise Euclidean distance matrices)
- output_test_<suffix>.txt (two-sample T-test and Wilcoxon rank test results from the local background model)
- PDF heatmaps of average Euclidean distance, contact frequency, loop candidates, and loop summits when -p 1

## How to apply

Create a Python 3.6.8 conda environment, pip install SnapFISH, then run 'SnapFISH call-loops' once per haplotype on its 3D coordinate file (tab-separated chr, cell_id, pos, x, y, z in nm) with a per-allele '-d' suffix (e.g., 129, CAST) so outputs do not overwrite, plus a 1D annotation file mapping each 5 Kb bin to its chr3 genomic location. For each allele, SnapFISH computes pairwise Euclidean distances across cells, averages them, and performs a two-sample T-test of each segment pair against a local background model built from pairs within a 25-50 Kb 1D genomic-distance square; loop candidates require T < 0 and FDR < 0.1 (default '-h 0.1'), with a Wilcoxon rank-test alternative ('-m w') if distances deviate from normality. Nearby candidates are grouped into clusters, singletons removed, and the summit with minimal FDR kept; the final loop list uses cluster summits with population-level contact frequency >=1/3 and singletons >=1/2. Run with '-p 1' to save heatmaps of average Euclidean distance, contact frequency, loop candidates, and loop summits. Judge success by candidate counts matching 44 (CAST) and 61 (129) and by a single summit per haplotype whose anchors fall within 10 Kb of the Sox2 promoter-super-enhancer anchors, consistent with the HiCCUPS loop from mESC bulk Hi-C data.

## Related tools

- **SnapFISH** (Primary pipeline: 'SnapFISH call-loops' run once per allele with a distinct '-d' suffix computes pairwise Euclidean distances, performs the two-sample T-test against the 25-50 Kb local background, and groups loop candidates into clusters to output per-haplotype summits) — https://github.com/HuMingLab/SnapFISH
- **HiCCUPS** (Orthogonal bulk Hi-C loop caller whose mESC HiCCUPS loop serves as the reference confirming the per-haplotype SnapFISH summit matches the known Sox2 promoter-super-enhancer loop)

## Examples

```
SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129
```

## Evaluation signals

- Loop candidate counts per haplotype match the reported 44 (CAST) and 61 (129)
- A single loop summit per haplotype falls between the Sox2 promoter and its super-enhancer, with anchors within 10 Kb of the Sox2 promoter-super-enhancer anchors in both CAST and 129 alleles
- All candidate pairs satisfy T < 0 and FDR < 0.1; final summits have population-level contact frequency >=1/3 (singletons >=1/2)
- Console output reports a plausible cutoff (average Euclidean distance between bin pairs 25 Kb apart; demo value '129chr3 cutoff: 349.4659' nm) and a strongly negative Pearson's r between the average distance and contact frequency matrices (demo -0.9845)
- Per-allele output files remain distinct (output_loop_candidate_129 vs CAST) — distinct '-d' suffixes prevented overwriting

## Limitations

- Sensitivity can be further improved, in particular when applying SnapFISH to DNA seqFISH+ data in mESCs
- Missing 3D coordinates and missing Euclidean distances are skipped rather than imputed; imputation may increase sample size and statistical power but is not implemented
- Only pair-wise chromatin interactions are considered; multi-way interactions are not detected
- The CAST allele contains a 7.5 Kb 4CBS insertion not used in the demo, and per-allele runs must use distinct '-d' suffixes or output files overwrite each other

## Evidence

- [abstract] SnapFISH can identify known chromatin loops from mouse embryonic stem cells with high sensitivity and accuracy.: "SnapFISH can identify known chromatin loops from mouse embryonic stem cells with high sensitivity and accuracy."
- [discussion] Local background neighborhood definition: "we define the local neighborhood of a given targeted segment pair as all of the identified pairs that fall within a square with 25 Kb ~ 50 Kb in 1D genomic distance"
- [discussion] Loop candidate threshold: "(T < 0) and FDR < 10%"
- [discussion] Final loop list contact frequency thresholds: "The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2"
- [discussion] Validation overlap definition for Sox2 enhancer-promoter anchors: "if and only if the SnapFISH loop anchors are within 10 Kb of Sox2 promoter-super-enhancer loop anchors"
- [readme] Per-allele command-line invocation with distinct suffix: "SnapFISH call-loops -i PATH/TO/3DCOOR.txt -o OUT/DIR -a PATH/TO/ANN.txt -p 1 -d SUFFIX"
- [readme] Allele-specific dataset caveat (CAST 4CBS insertion): "The CAST allele contains a 7.5Kb 4CBS insertion (data not used in this GitHub demo), while the 129 allele does not contain this insertion (data used in this GitHub demo)."
- [readme] Default FDR filtering threshold: "-h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1."
