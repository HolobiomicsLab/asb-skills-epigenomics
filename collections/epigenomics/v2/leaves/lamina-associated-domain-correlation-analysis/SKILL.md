---
name: lamina-associated-domain-correlation-analysis
description: Use when Hi-C-derived sub-compartment scores (e.g. CscoreTool-M *_cscore.txt
  outputs) need to be linked genome-wide to lamina-associated domains such as early-G1
  cTSA-seq LAD maps (GSE224282) to test which sub-compartment corresponds to lamina-proximal
  chromatin.
when_to_use_negative:
- Input Hi-C data lack a valid equal-length Rabl-position bed file, since CscoreTool-M
  requires the relative centromere-to-telomere position per window to estimate the
  Rabl effect
- The analysis targets single-compartment A/B assignments rather than multiple probabilistic
  sub-compartments within a cell population
- Comparing LADs across cell types without sub-compartment scores, where direct LAD
  overlap analysis rather than CscoreTool-M score correlation is appropriate
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CscoreTool-M
  role: Infers multiple sub-compartment probability scores from Hi-C summary data;
    its *_cscore.txt outputs are correlated with early-G1 cTSA-seq LADs
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 when the prebuilt
    CentOS executable does not work
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:1339fc8603d52c5026e1a85667b5ff2bf1e4e44520888698ea514e769007a035
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# lamina-associated-domain-correlation-analysis

## Summary

Correlates CscoreTool-M sub-compartment probability scores (from Hi-C) genome-wide with lamina-proximal chromatin maps such as early-G1 cTSA-seq LADs, to identify the sub-compartment capturing early-G1 LAD patterns (e.g. HCT116 C10_7, R = 0.42).

## When to use

Use when Hi-C-derived sub-compartment scores (e.g. CscoreTool-M *_cscore.txt outputs) need to be linked genome-wide to lamina-associated domains such as early-G1 cTSA-seq LAD maps (GSE224282) to test which sub-compartment corresponds to lamina-proximal chromatin.

## When NOT to use

- Input Hi-C data lack a valid equal-length Rabl-position bed file, since CscoreTool-M requires the relative centromere-to-telomere position per window to estimate the Rabl effect
- The analysis targets single-compartment A/B assignments rather than multiple probabilistic sub-compartments within a cell population
- Comparing LADs across cell types without sub-compartment scores, where direct LAD overlap analysis rather than CscoreTool-M score correlation is appropriate

## Inputs

- CscoreTool-M *_cscore.txt per-window sub-compartment probability scores for each cell type
- Hi-C summary file in HOMER runHiCpca.pl HiCsummary format (e.g. from Rao 2014 GM12878 or HCT116 Rao 2017 data)
- Equal-length genomic windows Rabl bed file with centromere-to-telomere relative position in column 4
- hg38_blacklisted.bed (ENCODE blacklist regions)
- Test_excluded.txt (translocation-mediated ExcludedInteractions regions)
- Early-G1 cTSA-seq LAD maps (GEO GSE224282)

## Outputs

- Genome-wide Pearson correlation (R, P-value) between a candidate sub-compartment score (e.g. HCT116 C10_7) and early-G1 cTSA-seq LAD mapping per genomic window
- Identification of the sub-compartment representing lamina-proximal early-G1 chromatin
- Normalized cscore bedgraph files for visualization of sub-compartment probabilities

## How to apply

Run CscoreTool-M on each cell type's Hi-C summary file with an equal-length Rabl bed file (fourth column giving relative centromere-to-telomere position), setting N_subcompartment=10 for HCT116 (Rao 2017) and N_subcompartment=5 for cell-cycle-sorted scHi-C (Nagano 2017), with N_session=12, and pass hg38_blacklisted.bed and Test_excluded.txt to exclude ENCODE blacklist regions and translocation-mediated interactions. Take the *_cscore.txt output, where compartment scores represent the probability of each genomic window locating in a specific sub-compartment (c-scores normalized to add up to 1 per region). Identify the candidate lamina-associated sub-compartment (C10_7 in HCT116) and compute a genome-wide Pearson correlation of its score against early-G1 cTSA-seq LAD mapping per genomic window, expecting R = 0.42 (P < 10-16). Optionally validate that the identified sub-compartment is distinct from housekeeping (C10_2) and cell-type-specific (C10_1) sub-compartments via GO enrichment across GM12878, OSN, mESC, IMR90, NHEK, and HCT116.

## Related tools

- **CscoreTool-M** (Infers multiple sub-compartment probability scores from Hi-C summary data; its *_cscore.txt outputs are correlated with early-G1 cTSA-seq LADs) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 when the prebuilt CentOS executable does not work)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt
```

## Evaluation signals

- Pearson correlation between HCT116 C10_7 scores and early-G1 cTSA-seq LAD mapping is R ≈ 0.42 with P < 10-16
- Cscore values are normalized to add up to 1 for each genomic region in the cscore bedgraph output
- Both hg38_blacklisted.bed (ENCODE blacklist) and Test_excluded.txt (translocation interactions) were passed as 7th/8th arguments to the run
- N_subcompartment and N_session match the reference settings (10 and 12 for HCT116; 5 for Nagano 2017 cell-cycle-sorted scHi-C)
- Genes in the identified sub-compartment are distinct from housekeeping (C10_2) and cell-type-specific (C10_1) GO enrichment patterns

## Limitations

- The compartment scores are probabilities of sub-compartment localization and quantify heterogeneity within a cell population, not per-cell measurements
- The prebuilt executable was compiled on CentOS and may need recompilation (g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3) on other systems
- Regions with no reads are not shown in the cscore bedgraph output, so windows lacking Hi-C coverage cannot be correlated

## Evidence

- [abstract] By analyzing 10 sub-compartments, we found a sub-compartment containing chromatin potentially related to the early-G1 chromatin regions proximal to the nuclear lamina in HCT116 cells: "By analyzing 10 sub-compartments, we found a sub-compartment containing chromatin potentially related to the early-G1 chromatin regions proximal to the nuclear lamina in HCT116 cells"
- [abstract] The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment.: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment."
- [readme] XXX_cscore.txt This is the Cscore estimated for each genomic window.: "XXX_cscore.txt This is the Cscore estimated for each genomic window."
- [readme] ./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt"
