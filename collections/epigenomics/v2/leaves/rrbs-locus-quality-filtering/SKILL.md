---
name: rrbs-locus-quality-filtering
description: Use when preparing multi-sample RRBS methylation data for epiallele analysis
  and you need to drop poorly covered or incompletely observed loci (e.g. shallow
  read depth, few CpGs, sex chromosomes) before fitting epiallele models.
when_to_use_negative:
- Input is whole-genome bisulfite sequencing (WGBS) rather than RRBS — the depth and
  CpG-count thresholds were calibrated for reduced-representation loci
- The analysis is a single-locus, single-sample methylation summary with no multi-sample
  epiallele comparison, where the model's power-borrowing across tissue samples does
  not apply
- Reads cannot be phased between disjoint loci and the question requires inter-locus
  phasing — a stated limitation of the approach
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Implementation language for the bed package and the locus-filtering workflow
    (read-depth, CpG-count, missingness, and chromosome filters applied in R)
  repo: https://github.com/james-e-barrett/bed
- name: bed
  role: Bayesian Epiallele Detection R package whose bed_marginal output supplies
    the per-locus read depth, epiallele frequencies, and phi_s vectors that the filters
    are applied to
  repo: https://github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub prior to filtering (install_github)
  repo: https://github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:957865b3e22b0a6bf4b5ffa773c90d47e4376b50a2bc49b6c4a6e9c3b75dfe1d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed
  - devtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# rrbs-locus-quality-filtering

## Summary

Quality-control filtering of RRBS loci prior to Bayesian epiallele detection, retaining only loci with sufficient read depth, CpG count, data completeness, and autosomal location so that downstream epiallele composition comparisons are statistically reliable.

## When to use

Use when preparing multi-sample RRBS methylation data for epiallele analysis and you need to drop poorly covered or incompletely observed loci (e.g. shallow read depth, few CpGs, sex chromosomes) before fitting epiallele models.

## When NOT to use

- Input is whole-genome bisulfite sequencing (WGBS) rather than RRBS — the depth and CpG-count thresholds were calibrated for reduced-representation loci
- The analysis is a single-locus, single-sample methylation summary with no multi-sample epiallele comparison, where the model's power-borrowing across tissue samples does not apply
- Reads cannot be phased between disjoint loci and the question requires inter-locus phasing — a stated limitation of the approach

## Inputs

- RRBS bisulfite sequencing methylation reads from multiple tissue samples (e.g. seven tumour regions plus one matched normal) organised per-locus
- Per-locus epiallele assignments and read depth from bed_fit / bed_marginal output (phi_s epiallele frequency distributions)
- Locus annotations including CpG count, read depth, and chromosome

## Outputs

- Filtered set of high-quality loci (median depth >= 100, >= 6 CpGs, <= 25% missing, autosomes only)
- Per-locus phi_s epiallele frequency distributions with epialleles below 5% of observed reads removed

## How to apply

After inferring per-locus epialleles with the bed package (bed_fit followed by bed_marginal marginalisation over w to obtain phi_s vectors), restrict analysis to loci passing all quality criteria: median read depth >= 100 across normal and tumour tissue samples, six or more CpGs per locus, no more than 25% missing data per locus, and discard any data from chromosomes X and Y. Additionally discard any epialleles that accounted for less than 5% of observed reads prior to computing phi_s. These thresholds ensure the Bayesian model has enough read-level evidence per locus and avoid sex-chromosome copy-number and methylation confounds; only loci occurring in two or more samples are carried forward for between-sample comparison.

## Related tools

- **R** (Implementation language for the bed package and the locus-filtering workflow (read-depth, CpG-count, missingness, and chromosome filters applied in R)) — https://github.com/james-e-barrett/bed
- **bed** (Bayesian Epiallele Detection R package whose bed_marginal output supplies the per-locus read depth, epiallele frequencies, and phi_s vectors that the filters are applied to) — https://github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub prior to filtering (install_github)) — https://github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed")
```

## Evaluation signals

- No retained locus has median read depth below 100 across the normal and tumour tissue samples
- Every retained locus has six or more CpGs and no more than 25% of data missing per locus
- No locus from chromosomes X or Y appears in the filtered locus set
- No epiallele representing less than 5% of observed reads at a locus survives into the phi_s distributions
- Filtered loci are restricted to those occurring in two or more tissue samples

## Limitations

- Reads cannot be phased between disjoint loci, so filtering and epiallele inference are strictly within-locus
- Study was limited to a single patient (one tumour with seven regions plus one matched normal); threshold robustness across larger multi-patient cohorts was deferred to future work
- Discarding epialleles under 5% of observed reads and requiring >= 25% data completeness may remove rare but biologically real subclonal epialleles

## Evidence

- [methods] Only loci with a median read depth >=100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [methods] We demanded that no more than 25% of data were missing per locus: "We demanded that no more than 25% of data were missing per locus"
- [methods] Any data from chromosomes X and Y were discarded.: "Any data from chromosomes X and Y were discarded."
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ s: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ"
- [readme] install_github("james-e-barrett/bed"): "To install this R package use the devtools install_github command"
