---
name: tumour-purity-decontamination
description: Use when RRBS-derived epiallele frequency distributions (phi_s vectors)
  from multiple tumour regions plus a matched normal are contaminated by normal tissue,
  and profiles must be purified before identifying loci that differ between normal
  and cancer tissue.
when_to_use_negative:
- Samples have no matched normal tissue available, since decontamination requires
  the normal epiallele distribution n and the (1-rho) contamination term
- Loci fail the study's quality filters (median read depth < 100, fewer than six CpGs,
  more than 25% missing data, or chromosomes X/Y), as phi_s is not reliable there
- The goal is phasing or comparing reads across disjoint loci, which the approach
  explicitly does not support
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2648
tools:
- name: bed
  role: R package implementing Bayesian Epiallele Detection (bed_fit, bed_marginal)
    that produces the epiallele profiles phi_s and supports the decontamination protocol
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github
  repo: github.com/james-e-barrett/bed
- name: ape
  role: Its 'fastme.bal' function builds phylogenetic trees from the decontaminated
    epiallele representations in downstream comparison of regions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:203153bab23bc7fd5c07fa2dc0968fc4bf6f81e7adb8e841ae76a4feaf8875a7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ape
  - bed
  - devtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tumour-purity-decontamination

## Summary

Estimates the fraction of normal tissue contamination (purity rho) in multi-region tumour samples and decontaminates the inferred epiallele profiles at each locus, so that downstream comparisons against matched normal epiallele composition reflect the pure tumour signal.

## When to use

Use when RRBS-derived epiallele frequency distributions (phi_s vectors) from multiple tumour regions plus a matched normal are contaminated by normal tissue, and profiles must be purified before identifying loci that differ between normal and cancer tissue.

## When NOT to use

- Samples have no matched normal tissue available, since decontamination requires the normal epiallele distribution n and the (1-rho) contamination term
- Loci fail the study's quality filters (median read depth < 100, fewer than six CpGs, more than 25% missing data, or chromosomes X/Y), as phi_s is not reliable there
- The goal is phasing or comparing reads across disjoint loci, which the approach explicitly does not support

## Inputs

- Per-locus epiallele frequency distributions phi_s for each tumour region and matched normal, obtained via bed_marginal marginalisation over w (loci filtered to median read depth >= 100, >= 6 CpGs, <= 25% missing data, chromosomes X/Y discarded, epialleles < 5% of reads removed)
- Matched normal tissue epiallele distribution n_q at each locus
- Per-tumour-region purity estimates rho derived from the empirical density of xi values

## Outputs

- Decontaminated tumour epiallele profiles t_q per locus and region, with values clipped to [0,1]
- Per-region tumour purity estimates rho
- Per-locus divergence statistics between decontaminated tumour and matched normal distributions, with significant loci written to significant_loci.tsv

## How to apply

First fit the Bayesian epiallele model (bed_fit) and marginalise over w (bed_marginal) to obtain per-locus epiallele frequency vectors phi_s, discarding epialleles contributing less than 5% of observed reads before computing phi_s. Estimate each tumour region's purity rho from the maximum value of the empirical density of xi, which reflects the degree of normal contamination. Then decontaminate each region's profile with t_q = (phi_q - (1-rho)*n_q)/rho, where n is the matched normal epiallele distribution, and clip any t values below 0 to 0 and any above 1 to 1. The resulting decontaminated t vectors are the inputs for locus-wise divergence comparisons against the normal profile and for phylogenetic tree construction using the 'fastme.bal' function of the ape R package.

## Related tools

- **bed** (R package implementing Bayesian Epiallele Detection (bed_fit, bed_marginal) that produces the epiallele profiles phi_s and supports the decontamination protocol) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github) — github.com/james-e-barrett/bed
- **ape** (Its 'fastme.bal' function builds phylogenetic trees from the decontaminated epiallele representations in downstream comparison of regions)

## Examples

```
install_github("james-e-barrett/bed"); t_q <- pmin(pmax((phi_q - (1 - rho) * n_q) / rho, 0), 1)
```

## Evaluation signals

- All decontaminated values t_q fall within [0,1] after the clipping rule (t<0 set to 0, t>1 set to 1)
- Estimated rho for each region corresponds to the maximum value of the empirical density of xi
- Decontaminated tumour profiles at flagged loci shift away from the matched normal composition, consistent with tumours moving towards less methylated epialleles
- significant_loci.tsv exists with one row per assessed locus, containing locus identifiers and per-region epiallele proportions
- Profiles computed only from epialleles retained after the 5%-of-reads discard rule at each locus

## Limitations

- The case study covers a single patient (one lung tumour, CRUK0062, seven regions plus one matched normal); larger multi-patient validation was deferred to future work
- Reads cannot be phased between disjoint loci, so epialleles and entropy may only be correlated between close loci
- Purity estimation depends on the shape of the empirical density of xi and can be unreliable at low depth or heavily contaminated regions; negative decontaminated values must be clipped to 0

## Evidence

- [abstract] The degree to which tumour samples are contaminated with normal tissue can be estimated and corrected for.: "The degree to which tumour samples are contaminated with normal tissue can be estimated and corrected for."
- [methods] Purity estimated from density of xi: "If we plot the empirical density of ξ values the parameter ρ can be estimated from the maximum value of ξ."
- [methods] Decontamination formula: "once estimates of ρ have been obtained we can calculate the ‘decontaminated’ tumour epiallele profiles at each locus"
- [methods] Clipping rule: [evidence span withheld — anchor status: not_found]
- [readme] README install instructions: "To install this R package use the devtools install_github command"
