---
name: gene-ontology-enrichment-of-sub-compartments
description: Use when genes have been assigned to CscoreTool-M sub-compartments (e.g.,
  C10_2 or C10_1 from a 10-sub-compartment run on Hi-C data) and you need to test
  whether these sub-compartments are enriched for housekeeping or cell-type-specific
  biological functions.
when_to_use_negative:
- Input is raw Hi-C summary data and sub-compartments have not yet been inferred —
  run CscoreTool-M first; this skill starts from cscore.txt assignments
- The goal is quantifying intra-population heterogeneity (Shannon information content
  over probability scores) rather than functional interpretation of sub-compartment
  gene content
- Genomic windows contain no genes or no reads — regions with no reads are not shown
  in the cscore outputs, leaving nothing to assign for GO analysis
edam_operation: http://edamontology.org/operation_3672
tools:
- name: CscoreTool-M
  role: Infers the multiple 3D genome sub-compartment probabilities whose gene assignments
    feed the GO enrichment analysis
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: HOMER
  role: Defines the HiCsummary input format that CscoreTool-M accepts for the upstream
    Hi-C interaction file
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:55188d8c0e41a1810a09bfff6c6c1f630b8c97d5d36d701c679eec54b97d1ae8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - HOMER
  - g++
  edam_operation: http://edamontology.org/operation_3672
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-ontology-enrichment-of-sub-compartments

## Summary

Annotates CscoreTool-M 3D genome sub-compartments by performing Gene Ontology enrichment on genes assigned to each sub-compartment, demonstrating that sub-compartments correspond to housekeeping or cell-type-specific functions. This validates the biological interpretability of probabilistic sub-compartment assignments.

## When to use

Use when genes have been assigned to CscoreTool-M sub-compartments (e.g., C10_2 or C10_1 from a 10-sub-compartment run on Hi-C data) and you need to test whether these sub-compartments are enriched for housekeeping or cell-type-specific biological functions.

## When NOT to use

- Input is raw Hi-C summary data and sub-compartments have not yet been inferred — run CscoreTool-M first; this skill starts from cscore.txt assignments
- The goal is quantifying intra-population heterogeneity (Shannon information content over probability scores) rather than functional interpretation of sub-compartment gene content
- Genomic windows contain no genes or no reads — regions with no reads are not shown in the cscore outputs, leaving nothing to assign for GO analysis

## Inputs

- CscoreTool-M output XXX_cscore.txt (sub-compartment probability scores per genomic window)
- Gene annotation mapping genomic windows to genes (e.g., hg38 windows from hg38_500k_Rabl.bed)
- Optional: hg38_blacklisted.bed (ENCODE blacklist regions) and Test_excluded.txt (translocation exclusions) used in the upstream CscoreTool-M run

## Outputs

- GO enrichment results for genes in each sub-compartment (e.g., C10_2 housekeeping terms, C10_1 cell-type-specific terms)
- Gene-to-sub-compartment assignment table across cell types (GM12878, OSN, mESC, IMR90, NHEK, HCT116)

## How to apply

First run CscoreTool-M on a Hi-C summary file (HOMER runHiCpca.pl HiCsummary format) with the Rabl bed file, e.g. N_subcompartment=10 for HCT116 (Rao 2017) data, passing hg38_blacklisted.bed and Test_excluded.txt to exclude ENCODE blacklist regions and translocation-mediated interactions. Take the *_cscore.txt output, which gives per-window probabilities of locating in each sub-compartment, and assign each genomic window (and its genes) to the sub-compartment with the highest score. Then perform Gene Ontology enrichment on the genes assigned to the sub-compartment of interest — e.g., C10_2 expected to be housekeeping and C10_1 expected to be cell-type-specific — across cell types such as GM12878, OSN, mESC, IMR90, NHEK, and HCT116. Judge success by whether significant GO term enrichment matching the expected functional character (housekeeping vs. cell-type-specific) is recovered, consistent with the article's finding that CscoreTool-M can identify sub-compartments enriched in these functions.

## Related tools

- **CscoreTool-M** (Infers the multiple 3D genome sub-compartment probabilities whose gene assignments feed the GO enrichment analysis) — https://github.com/scoutzxb/CscoreTool-M
- **HOMER** (Defines the HiCsummary input format that CscoreTool-M accepts for the upstream Hi-C interaction file)

## Evaluation signals

- Genes assigned to C10_2 sub-compartments show significant enrichment of housekeeping GO terms
- Genes assigned to C10_1 sub-compartments show significant enrichment of cell-type-specific GO terms
- Enrichment pattern is consistent across cell types analyzed (GM12878, OSN, mESC, IMR90, NHEK, HCT116)
- Upstream cscore.txt c-scores are normalized to add up to 1 for each genomic region, confirming valid sub-compartment probabilities before gene assignment

## Limitations

- Regions with no reads are not shown in the bedgraph output and yield no sub-compartment assignment, so genes in those windows cannot enter the enrichment analysis
- ENCODE blacklist regions (possibly due to copy number variations) and translocation-mediated interactions must be excluded upstream, or gene assignments in affected windows may be unreliable
- The housekeeping vs. cell-type-specific characterization was demonstrated for 10-sub-compartment analyses; other N_subcompartment settings may not yield functionally interpretable groups

## Evidence

- [abstract] Finally, we show that CscoreTool-M can identify sub-compartments that contain genes enriched in housekeeping or cell-type-specific functions.: "Finally, we show that CscoreTool-M can identify sub-compartments that contain genes enriched in housekeeping or cell-type-specific functions."
- [readme] XXX_cscore.txt This is the Cscore estimated for each genomic window.: "XXX_cscore.txt This is the Cscore estimated for each genomic window."
- [intro] We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl.: "We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl."
