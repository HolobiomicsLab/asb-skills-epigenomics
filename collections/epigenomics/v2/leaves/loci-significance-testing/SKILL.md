---
name: loci-significance-testing
description: Use when RRBS-derived per-locus epiallele frequency vectors (phi_s from
  bed_marginal) are available for matched normal and multi-region tumour samples and
  the goal is to find loci whose epiallele composition differs significantly between
  normal and cancer tissue.
when_to_use_negative:
- Reads are not phased across disjoint loci — do not interpret epiallele composition
  differences as spanning multiple distant CpGs (a stated limitation of the approach)
- The data come from a single locus failing the paper's filters (median depth < 100,
  fewer than 6 CpGs, >25% missing data, or chromosomes X/Y) — insufficient read support
  invalidates the comparison
- Tumour regions lack an estimated purity rho, so decontamination cannot be performed
  and raw phi_q profiles would be confounded by normal tissue contamination
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: R
  role: Implementation language for loading phi_s vectors, decontamination, and locus-level
    significance testing
  repo: https://github.com/james-e-barrett/bed
- name: bed
  role: Bayesian Epiallele Detection R package producing the epiallele fits and bed_marginal
    phi_s distributions the comparison consumes
  repo: https://github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:02d16148ca83548a5e6a5a2e653b9325a2cf83deff045971f08908ca8af31a39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/james-e-barrett/bed
  tools:
  - R
  - bed
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# loci-significance-testing

## Summary

Tests whether the decontaminated tumour epiallele composition differs significantly from the matched normal composition at individual CpG loci, flagging the loci where differences are significant. This operationalises the paper's goal of identifying epialleles that differ between normal and cancer tissue.

## When to use

Use when RRBS-derived per-locus epiallele frequency vectors (phi_s from bed_marginal) are available for matched normal and multi-region tumour samples and the goal is to find loci whose epiallele composition differs significantly between normal and cancer tissue.

## When NOT to use

- Reads are not phased across disjoint loci — do not interpret epiallele composition differences as spanning multiple distant CpGs (a stated limitation of the approach)
- The data come from a single locus failing the paper's filters (median depth < 100, fewer than 6 CpGs, >25% missing data, or chromosomes X/Y) — insufficient read support invalidates the comparison
- Tumour regions lack an estimated purity rho, so decontamination cannot be performed and raw phi_q profiles would be confounded by normal tissue contamination

## Inputs

- Per-locus epiallele frequency distributions (phi_s vectors) for matched normal and tumour regions, derived via bed_marginal marginalisation over w
- Estimated tumour purity rho per tumour region (from the xi density-maximum protocol)
- Locus metadata: median read depth, CpG count, fraction of missing data, chromosome

## Outputs

- significant_loci.tsv: one row per assessed locus with locus identifiers, per-region epiallele proportions, and the normal-vs-tumour comparison statistic
- Flagged list of loci with significant normal-vs-cancer epiallele composition differences

## How to apply

Load per-locus epiallele frequency distributions (phi_s vectors obtained by marginalising bed_marginal output over w) restricted to loci passing the paper's filters: median read depth >= 100 across normal and tumour samples, six or more CpGs, no more than 25% missing data, chromosomes X/Y discarded, and epialleles accounting for less than 5% of observed reads removed prior to computing phi_s. For each tumour region, decontaminate the observed epiallele profile using t_q = (phi_q - (1-rho)*n_q)/rho with the region's estimated purity rho, clipping values outside [0,1]. For each locus, compute a divergence statistic between the decontaminated tumour composition t and the matched normal distribution n, and flag loci where the difference is significant. Write flagged loci, per-region epiallele proportions, and the comparison statistic to significant_loci.tsv. The rationale is that purity correction removes normal-tissue contamination so that remaining shifts (typically towards less methylated epialleles) reflect genuine tumour-specific epiallele changes.

## Related tools

- **R** (Implementation language for loading phi_s vectors, decontamination, and locus-level significance testing) — https://github.com/james-e-barrett/bed
- **bed** (Bayesian Epiallele Detection R package producing the epiallele fits and bed_marginal phi_s distributions the comparison consumes) — https://github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed")  # then use ?bed_fit and bed_marginal to obtain phi_s before testing loci
```

## Evaluation signals

- significant_loci.tsv exists and contains one row per assessed locus with locus identifiers and per-region epiallele proportions
- Flagged loci show tumour epiallele compositions shifted away from the matched normal profile, consistent with the paper's qualitative finding that tumour samples shift towards less methylated epialleles
- All assessed loci satisfy the filter criteria: median read depth >= 100, six or more CpGs, no more than 25% missing data, chromosomes X/Y excluded
- Decontaminated proportions t_q lie within [0,1] after clipping, and epialleles below the 5% read-abundance cutoff were discarded before computing phi_s

## Limitations

- Reads cannot be phased between disjoint loci, so epiallele-level disorder and differences may be correlated but not linked across close loci
- The case study is limited to a single patient (one tumour, CRUK0062, seven regions plus one matched normal); larger multi-patient studies were deferred to future work
- Cases where the decontamination formula yields t^mu < 0 or > 1 must be clipped, indicating unstable estimates at low purity or high contamination

## Evidence

- [abstract] By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour, identify epialleles that differ between normal and cancer tissue: "By tracing the distribution of epialleles throughout the tumour we can infer the phylogenetic history of the tumour, identify epialleles that differ between normal and cancer tissue"
- [methods] Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered.: "Only loci with a median read depth ≥100 across normal and tumour tissue samples and six or more CpGs were considered."
- [methods] Any cases where t^μ<0 are set to 0 and any cases where t^μ>1 are set to 1.: [evidence span withheld — anchor status: not_found]
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕs: [evidence span withheld — anchor status: not_found]
- [readme] install_github("james-e-barrett/bed"): "To install this R package use the devtools install_github command:"
