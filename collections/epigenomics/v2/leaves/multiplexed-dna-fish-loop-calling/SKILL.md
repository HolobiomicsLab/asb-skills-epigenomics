---
name: multiplexed-dna-fish-loop-calling
description: Use when analyzing 3D chromatin tracing coordinates (per-cell x/y/z per
  locus) to call chromatin loops with SnapFISH, or to test how detection-efficiency
  stratification affects loop-calling sensitivity and false positives.
when_to_use_negative:
- Input is proximity-ligation assay data (Hi-C/Micro-C contact counts) rather than
  per-cell 3D imaging coordinates — existing count-based loop callers, not SnapFISH,
  apply
- The goal is detecting multi-way (higher-order) chromatin interactions — SnapFISH
  only considers pair-wise interactions
- The data are DNA seqFISH+ mESC datasets where the authors note sensitivity can be
  further improved, if high-sensitivity de novo calling is required
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: SnapFISH
  role: 'Core pipeline: computes pairwise/average Euclidean distances, runs the local-background
    two-sample T-test, and calls loop candidates, summits, and final loops'
  repo: https://github.com/HuMingLab/SnapFISH
- name: pandas
  role: Splits the 3D coordinate file into allele- and detection-efficiency-group
    subsets
- name: scipy
  role: Two-sample T-test used by the local background model on each loci pair
- name: statsmodels
  role: FDR computation for filtering loop candidates at FDR < 0.1
- name: numpy
  role: Euclidean distance computation across 3D locus coordinates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:08d1fd2496b4f0096bbd973214543ab6edee29406b223ea7c6a0c0dc4638543e
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
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multiplexed-dna-fish-loop-calling

## Summary

Reproduce SnapFISH-based chromatin loop calling on multiplexed DNA FISH (chromatin tracing) data, stratified by allele and targeted segment detection efficiency, to quantify how low detection efficiency inflates false-positive loop calls (e.g., 3 false positives for the weak-loop CAST allele at ~57%/55% detection).

## When to use

Use when analyzing 3D chromatin tracing coordinates (per-cell x/y/z per locus) to call chromatin loops with SnapFISH, or to test how detection-efficiency stratification affects loop-calling sensitivity and false positives.

## When NOT to use

- Input is proximity-ligation assay data (Hi-C/Micro-C contact counts) rather than per-cell 3D imaging coordinates — existing count-based loop callers, not SnapFISH, apply
- The goal is detecting multi-way (higher-order) chromatin interactions — SnapFISH only considers pair-wise interactions
- The data are DNA seqFISH+ mESC datasets where the authors note sensitivity can be further improved, if high-sensitivity de novo calling is required

## Inputs

- 3D_COOR.txt: tab-delimited file with columns chr, cell_id, pos, x, y, z (3D coordinates in nm) for each bin in each cell
- ANN.txt: 1D genomic annotation file with columns chr, start, end, pos for each bin (5Kb bins, mm10 chr3:34,601,078–34,806,078 Sox2 locus)
- Per-allele detection-efficiency assignments used to partition cells into high/median/low groups

## Outputs

- output_3D_dist_<SUFFIX>.txt: pairwise Euclidean distances between bins in each cell
- output_3D_dist_<SUFFIX>_avg.txt: average pairwise distance matrix across cells
- output_test_<SUFFIX>.txt: T-test and Wilcoxon rank test results from the local background model
- output_loop_candidate_<SUFFIX>.txt and output_loop_summit_<SUFFIX>.txt: loop candidates and summits
- Per-subset loop call comparison flagging false positives (expected: 3 in low-efficiency CAST, none in any 129 group)

## How to apply

Split the 3D coordinate file into six subsets by allele (CAST, 129) and detection-efficiency group (high ~85.4%/82.6%, median ~73.2%/70.4%, low ~57.0%/55.0%) with pandas, keeping 1416 alleles per allele type. For each subset run `SnapFISH call-loops` on the tab-delimited 3D coordinate file (chr, cell_id, pos, x, y, z in nm) and the 1D genomic annotation file (chr, start, end, pos) to compute pairwise Euclidean distances per cell and the average distance matrix. SnapFISH applies a local background model over segment pairs within 25–50 Kb 1D genomic distance and a two-sample T-test (scipy/statsmodels), keeping candidates with T < 0 and FDR < 0.1 (the `-h 0.1` default), then groups nearby candidates into clusters, keeps summits, and filters by population-level contact frequency (summits >= 1/3, singletons >= 1/2). Finally, compare called loops across the six groups, checking overlap with Sox2 enhancer-promoter anchors within 10 Kb, and flag any extra calls as false positives.

## Related tools

- **SnapFISH** (Core pipeline: computes pairwise/average Euclidean distances, runs the local-background two-sample T-test, and calls loop candidates, summits, and final loops) — https://github.com/HuMingLab/SnapFISH
- **pandas** (Splits the 3D coordinate file into allele- and detection-efficiency-group subsets)
- **scipy** (Two-sample T-test used by the local background model on each loci pair)
- **statsmodels** (FDR computation for filtering loop candidates at FDR < 0.1)
- **numpy** (Euclidean distance computation across 3D locus coordinates)

## Examples

```
SnapFISH call-loops -i ext/129_3d_coor.txt -o tmp -a ext/input_ann.txt -p 1 -d 129
```

## Evaluation signals

- Six subsets created: two alleles (CAST, 129) × three detection-efficiency groups, each with 1416 alleles per allele type
- SnapFISH prints a cutoff (average Euclidean distance in nm between bin pairs 25 Kb apart, e.g. 349.4659) and Pearson's r between average distance and contact frequency (e.g. -0.9845), confirming correct parsing
- Loop candidates all satisfy T < 0 and FDR < 0.1 under the local background model (segment pairs within 25–50 Kb 1D distance)
- Low-efficiency CAST group contains exactly 3 false-positive loop calls and no false positives appear in any 129 group
- Called loops overlap Sox2 enhancer-promoter anchors only if anchors are within 10 Kb of the promoter-super-enhancer loop anchors

## Limitations

- Only pair-wise chromatin interactions are considered; multi-way interactions are not detected
- Sensitivity can be further improved, particularly for DNA seqFISH+ data in mESCs
- Imputation of missing 3D coordinates and missing Euclidean distances is proposed but not implemented, limiting statistical power
- No simulator exists for generating realistic multiplexed DNA FISH data, complicating method evaluation

## Evidence

- [discussion] we define the local neighborhood of a given targeted segment pair as all of the identified pairs that fall within a square with 25 Kb ~ 50 Kb in 1D genomic distance: "we define the local neighborhood of a given targeted segment pair as all of the identified pairs that fall within a square with 25 Kb ~ 50 Kb in 1D genomic distance"
- [discussion] The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2: "The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2"
- [readme] -h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1.: "-h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1."
- [readme] 41 loci cover a 205Kb region in mESC Sox2 locus - chr3:34,601,078–34,806,078 (ref: mm10). This dataset contains two alleles: CAST and 129.: "41 loci cover a 205Kb region in mESC Sox2 locus - chr3:34,601,078–34,806,078 (ref: mm10). This dataset contains two alleles: CAST and 129."
- [methods] Step 1a: Take 3D coordinates of each locus in each cell as input, calculate Eucildean disance between any two loci pairs: "Step 1a: Take 3D coordinates of each locus in each cell as input, calculate Eucildean disance between any two loci pairs"
