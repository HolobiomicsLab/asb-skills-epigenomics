---
name: genomic-interval-overlap-quantification
description: Use when two or more sets of genomic intervals (e.g. DMRs from different
  callers, i-DMRs vs reference regions) must be intersected to compute overlap fractions,
  or when CpG-level methylation records need region names from a BED annotation file
  before running DXM.
when_to_use_negative:
- Region names are already unique and embedded in the methylation input file, so no
  annotation intersect is needed before DXM
- The analysis needs per-CpG methylation-difference statistics rather than coordinate-level
  interval overlap between DMR sets
- Intervals come from different genome builds or coordinate systems, where naive intersect
  would produce spurious overlaps
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: bedtools
  role: Intersects interval sets with -wo to attach CGI/annotation region names to
    DXM input and to compute DMR/i-DMR coordinate overlaps
  repo: https://bedtools.readthedocs.io/en/latest/
- name: DXM
  role: Produces the i-DMR interval set via dxm_solveMethylation and dxm_callIDMR
    whose *_DXM_dmrs.txt coordinates are intersected against DSS DMRs
  repo: https://github.com/edwardslab-wustl/dxm
- name: DSS
  role: Calls DMRs independently on the sorted GCB vs monocyte methylation counts
    (delta = 0.3, p < 0.05) to form the reference interval set
- name: python3
  role: Computes per-region and summary concordance fractions from the bedtools intersect
    output and writes dss_idmr_overlap.tsv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab516
    title: Determining subpopulation methylation profiles from bisulfite sequencing
      data of heterogeneous samples using DXM
  source_content_hash: sha256:ba9e663de0966f7904a1e13246e2be59aa44406dbc939138099645e228f2c99a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DSS
  - DXM
  - bedtools
  - python3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Genomic interval overlap quantification

## Summary

Uses bedtools intersect to quantify coordinate-level agreement between genomic interval sets — here, DXM intrasample DMRs (i-DMRs) versus DSS-called DMRs from sorted GCB vs monocyte bisulfite sequencing data, and to attach region names/annotations (e.g. CGI IDs) to DXM input regions.

## When to use

Use when two or more sets of genomic intervals (e.g. DMRs from different callers, i-DMRs vs reference regions) must be intersected to compute overlap fractions, or when CpG-level methylation records need region names from a BED annotation file before running DXM.

## When NOT to use

- Region names are already unique and embedded in the methylation input file, so no annotation intersect is needed before DXM
- The analysis needs per-CpG methylation-difference statistics rather than coordinate-level interval overlap between DMR sets
- Intervals come from different genome builds or coordinate systems, where naive intersect would produce spurious overlaps

## Inputs

- DSS-called DMR coordinates from sorted GCB vs monocyte bisulfite sequencing data (bed-like interval file)
- DXM i-DMR coordinates from the *_DXM_dmrs.txt output of dxm_callIDMR
- Methylation records in tab-delimited BED-like format (chromosome, position1, position2, methylation_level, coverage)
- Annotation BED file of regions with names (e.g. cgi.bed from UCSC Genome Browser Table Browser, or refGene_hg19_10kb_fixed.txt.gz)

## Outputs

- DXM input file (dxm_in.bed) with per-region unique region names attached via bedtools intersect
- Per-region overlap table and summary concordance fractions written to dss_idmr_overlap.tsv

## How to apply

Convert each interval set to a BED-like file; for DXM input prep, intersect methylation records with an annotation BED (e.g. the UCSC Table Browser cgi.bed or refGene_hg19_10kb_fixed.txt.gz) using 'bedtools intersect -wo' and an awk statement that keeps the annotation ID as the unique region name column required by DXM. For concordance assessment, run bedtools intersect between DSS DMR coordinates (DSS run with equal dispersion, delta = 0.3, p-value < 0.05) and DXM i-DMR coordinates from the *_DXM_dmrs.txt file, then compute per-region overlap in python3 and write per-region and summary fractions (e.g. fraction of i-DMRs overlapping a DSS DMR; recovery of DSS-'missed' DMRs under refined region definitions) to dss_idmr_overlap.tsv. Rationale: caller-level differences between DSS-called DMRs and DXM i-DMRs are expected and comparable to known variability across DMR callers, so the overlap fraction (~64% in simulated GCB-monocyte mixtures, with ~86% of 'missed' DMRs recovered) is the validation metric, not exact identity.

## Related tools

- **bedtools** (Intersects interval sets with -wo to attach CGI/annotation region names to DXM input and to compute DMR/i-DMR coordinate overlaps) — https://bedtools.readthedocs.io/en/latest/
- **DXM** (Produces the i-DMR interval set via dxm_solveMethylation and dxm_callIDMR whose *_DXM_dmrs.txt coordinates are intersected against DSS DMRs) — https://github.com/edwardslab-wustl/dxm
- **DSS** (Calls DMRs independently on the sorted GCB vs monocyte methylation counts (delta = 0.3, p < 0.05) to form the reference interval set)
- **python3** (Computes per-region and summary concordance fractions from the bedtools intersect output and writes dss_idmr_overlap.tsv)

## Examples

```
bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed
```

## Evaluation signals

- The computed i-DMR/DMR concordance reproduces the reported ~64% agreement between DSS-called DMRs and DXM i-DMRs on simulated GCB-monocyte mixtures
- Recovery of ~86% of DSS-'missed' DMRs under refined region definitions is confirmed before accepting the result
- Every DXM input region in dxm_in.bed carries a unique region name from the annotation BED (e.g. CGI ID), as required by DXM
- Overlap table columns are consistent with bedtools intersect -wo output column ordering after the awk column selection
- Fraction metrics lie in [0,1] and per-region rows sum consistently with the reported summary fractions

## Limitations

- Differences between DSS-called DMRs and DXM i-DMRs are expected and similar to variability across different DMR-callers, so concordance is not 1:1 identity
- DXM ignores chromosome and position2 internally and computes across all CpGs of a named region, so overlap results depend on how region definitions (e.g. CGI vs TSS±5kb) were chosen
- Overlapping intervals with multiple putative iDMRs sharing the same region name must be handled when counting overlaps

## Evidence

- [readme] bedtools intersect for DXM region naming: "bedtools intersect -wo -a methylation.bed -b cgi.bed | awk '{print $1"\t"$2"\t"$3"\t"$9"\t"$4"\t"$5;}' > dxm_in.bed"
- [readme] region names via bedtools: "We recommend adding region names with utilities such as the 'intersect' command from [bedtools]"
- [discussion] DMR-caller variability rationale: "The differences between DSS-called DMRs and i-DMRs in artificial mixtures are similar to those observed between different DMR-callers (62)."
- [readme] multiple iDMRs share region name: "If there are multiple putative iDMRs for the same region, they will have the same corresponding region name."
