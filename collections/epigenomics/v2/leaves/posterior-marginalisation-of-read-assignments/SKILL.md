---
name: posterior-marginalisation-of-read-assignments
description: Use when RRBS reads at a multi-sample locus have been assigned to epialleles
  by bed_fit and you need per-sample epiallele distributions (phi_s) that account
  for assignment uncertainty rather than hard MAP read labels.
when_to_use_negative:
- Reads cannot be phased between disjoint loci, so this skill is inappropriate when
  the goal is to link epialleles across separate loci within a read
- Loci with median read depth < 100, fewer than six CpGs, more than 25% missing data,
  or on chromosomes X/Y should not enter marginalisation
- If only hard MAP assignments are required (no propagation of assignment uncertainty),
  bed_fit output alone suffices without bed_marginal
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: bed_marginal
  role: R function from the bed package that computes the posterior marginal over
    w with other parameters fixed at MAP values, used to derive per-sample epiallele
    distributions phi_s
  repo: github.com/james-e-barrett/bed
- name: bed_fit
  role: Upstream R function that produces the MAP estimates of epiallele assignments
    w, epiallele patterns X, and noise epsilon that bed_marginal consumes
  repo: github.com/james-e-barrett/bed
- name: devtools
  role: Used to install the bed package from GitHub via install_github before running
    bed_marginal
  repo: github.com/james-e-barrett/bed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-017-1753-2
    title: Quantification of tumour evolution and heterogeneity via Bayesian epiallele
      detection
  source_content_hash: sha256:219ffdbe5d167c0ee2b446211dd5aab29789f4e8bde797d967d04c6a34b86210
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - bed_fit
  - bed_marginal
  - devtools
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Posterior marginalisation of read assignments

## Summary

Within the Bayesian epiallele detection (BED) framework, uncertainty about which epiallele an observed RRBS sequencing read originated from is explicitly incorporated by fixing all other parameters to their MAP values and marginalising over the posterior density of the read-to-epiallele assignment vector w, using the R function bed_marginal.

## When to use

Use when RRBS reads at a multi-sample locus have been assigned to epialleles by bed_fit and you need per-sample epiallele distributions (phi_s) that account for assignment uncertainty rather than hard MAP read labels.

## When NOT to use

- Reads cannot be phased between disjoint loci, so this skill is inappropriate when the goal is to link epialleles across separate loci within a read
- Loci with median read depth < 100, fewer than six CpGs, more than 25% missing data, or on chromosomes X/Y should not enter marginalisation
- If only hard MAP assignments are required (no propagation of assignment uncertainty), bed_fit output alone suffices without bed_marginal

## Inputs

- RRBS read matrix Y at each eligible locus (pooled across all tissue samples: seven tumour regions of CRUK0062 plus one matched normal)
- MAP-estimated epiallele patterns X from bed_fit
- MAP-estimated read-to-epiallele assignment matrix W from bed_fit
- Number of epialleles Q selected by minimum AIC and retained epialleles (>= 5% of observed reads)

## Outputs

- Posterior marginal distribution over w_i for each observed read
- Per-sample epiallele distributions phi_s (proportion of reads per sample attributed to each retained epiallele)
- Downstream quantities derived from phi_s: sample purity estimates, decontaminated tumour epiallele profiles, phylogenetic tree input, and epiallele Shannon entropy per locus

## How to apply

After bed_fit(Y) converges to MAP estimates of the epiallele assignments w, epiallele patterns X, and noise epsilon, call bed_marginal(Y, X, W) on the pooled reads from all seven tumour regions plus the matched normal sample at each eligible locus (median read depth >= 100, six or more CpGs, <= 25% missing data, chromosomes X/Y excluded). bed_marginal computes the marginal density over each w_i by fixing all other parameters (X, epsilon, Q) to their MAP values, yielding a posterior distribution over which epiallele each read originated from. Aggregate these posteriors into per-sample epiallele distributions phi_s, i.e. the proportion of reads per sample attributed to each retained epiallele, after discarding inferred epialleles accounting for less than 5% of observed reads. The rationale is that hard assignments discard uncertainty: marginalising over w propagates it into downstream quantities such as phi_s, sample purity, phylogenetic tree construction, and epiallele Shannon entropy. Save the resulting phi_s vectors (with X, Q, epsilon) as the inference output, e.g. epiallele_inference_results.rds.

## Related tools

- **bed_marginal** (R function from the bed package that computes the posterior marginal over w with other parameters fixed at MAP values, used to derive per-sample epiallele distributions phi_s) — github.com/james-e-barrett/bed
- **bed_fit** (Upstream R function that produces the MAP estimates of epiallele assignments w, epiallele patterns X, and noise epsilon that bed_marginal consumes) — github.com/james-e-barrett/bed
- **devtools** (Used to install the bed package from GitHub via install_github before running bed_marginal) — github.com/james-e-barrett/bed

## Examples

```
library(devtools); install_github("james-e-barrett/bed"); fit <- bed_fit(Y); marg <- bed_marginal(Y, fit$X, fit$W); phi_s <- compute_phi_s(marg)
```

## Evaluation signals

- The MAP fit feeding marginalisation must have converged (typically two or three iterations) and correspond to the minimum AIC(Q) = -2 log p(Y|X,w,epsilon,Q) + 2Qd across candidate Q values
- Retained epialleles in phi_s must each account for at least 5% of observed reads; smaller ones must be discarded before computing phi_s
- For each sample s, the per-sample epiallele distribution phi_s must be a valid proportion vector (components in [0,1], attributable reads partitioned among retained epialleles)
- The noise MAP estimate epsilon = alpha0/(alpha0+alpha1) should lie in (0,1) and be consistent with the fit used for marginalisation
- Downstream sanity check: decontaminated epiallele proportions from phi_s must be clipped to [0,1] per the paper's protocol

## Limitations

- Reads cannot be phased between disjoint loci; the number of epialleles and entropy may only be correlated between close loci
- The approach was validated on a single patient (one tumour with seven regions plus one matched normal); larger multi-patient studies were deferred to future work
- Uncertainty quantification depends on MAP estimates of X, epsilon, and Q being correct; these are fixed rather than jointly inferred during marginalisation

## Evidence

- [abstract] Uncertainty as to which epiallele an observed sequencing read originated from is explicitly incorporated by marginalising over the appropriate posterior densities.: "Uncertainty as to which epiallele an observed sequencing read originated from is explicitly incorporated by marginalising over the appropriate posterior densities."
- [methods] The marginal density over w_i is given by fixing all other parameters to their MAP values: "The Bayesian approach allows this uncertainty to be incorporated into our analysis. The marginal density over w
i is given by fixing all other parameters to their MAP values"
- [methods] any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ
s: "any epialleles that accounted for less than 5% of observed reads were discarded prior to the computation of ϕ
s"
- [readme] Use the help function to find an example of how to use use the model:: "Use the help function to find an example of how to use use the model:

    ?bed_fit"
