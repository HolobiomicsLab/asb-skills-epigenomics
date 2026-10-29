---
name: count-distribution-maximum-likelihood-fitting
description: Use when you need to justify or select an emission/count distribution
  (GP, ZIP, Poisson, NB) for binned sequencing read counts, e.g. before configuring
  an HMM peak-caller on 25 bp bin HDF counts.
when_to_use_negative:
- Input is continuous or non-count data (e.g. ChIP-chip intensity probes) rather than
  discrete per-bin read/HDF counts
- Reads have not been converted into HDF coverage per fixed-length bins — fit nothing
  until the 25 bp binning and HMM bin classification step is done
- The goal is simply calling peaks rather than validating the underlying count model;
  use HPeak's peak-calling output directly instead
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HPeak
  role: Partitions the genome into 25 bp bins, counts HDFs per bin, and classifies
    bins as ChIP-enriched or background via a two-state HMM with GP/ZIP emissions,
    providing the count data and region classes to fit
  repo: https://github.com/cauyrd/HPeak
- name: scipy.stats
  role: Python library used to fit the GP, ZIP, Poisson, and negative binomial distributions
    to HDF counts by maximum likelihood
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:97cfa2465b2712db1169131003273572c03210cf56d6e2c13f0d479b3b7e0f41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - HPeak
  - scipy.stats
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# count-distribution-maximum-likelihood-fitting

## Summary

Fit candidate discrete count distributions (generalized Poisson, zero-inflated Poisson, Poisson, negative binomial) by maximum likelihood to HDF counts in ChIP-enriched vs background bins and rank them by a goodness-of-fit criterion. This justifies HPeak's choice of GP/ZIP emissions in its two-state HMM for ChIP-Seq peak calling.

## When to use

Use when you need to justify or select an emission/count distribution (GP, ZIP, Poisson, NB) for binned sequencing read counts, e.g. before configuring an HMM peak-caller on 25 bp bin HDF counts.

## When NOT to use

- Input is continuous or non-count data (e.g. ChIP-chip intensity probes) rather than discrete per-bin read/HDF counts
- Reads have not been converted into HDF coverage per fixed-length bins — fit nothing until the 25 bp binning and HMM bin classification step is done
- The goal is simply calling peaks rather than validating the underlying count model; use HPeak's peak-calling output directly instead

## Inputs

- ChIP-Seq read alignments (BED or ELAND format) for treatment and control samples (e.g. NRSF or STAT1 datasets)
- Per-bin HDF counts (25 bp bins) from HPeak genome partitioning
- Two-state HMM bin classification (ChIP-enriched vs background) from HPeak

## Outputs

- Maximum-likelihood parameter estimates for GP, ZIP, Poisson, and NB fits per region class
- Ranked distribution-fit comparison table (e.g. distribution_fit_comparison.csv) with AIC/log-likelihood per candidate
- Justification of the emission distributions (GP for enriched, ZIP for background) used by the HMM

## How to apply

Using HPeak, partition the genome into 25 bp bins and count hypothetical DNA fragments (HDFs) per bin for the ChIP-Seq dataset (extend each short read directionally from its start position to form HDFs). Split bins into ChIP-enriched and background classes according to HPeak's two-state HMM (GP/ZIP emissions) classification. Fit the GP, Poisson, and negative binomial distributions to the enriched-bin HDF counts by maximum likelihood (e.g. scipy.stats in Python), and fit the ZIP, Poisson, and NB distributions to the background-bin counts. Compare fitted distributions per dataset and region type using a goodness-of-fit criterion such as AIC or log-likelihood, and rank candidates. The expected outcome, validating the procedure, is that GP is best for enriched bins (much better than Poisson, slightly better than NB) and ZIP is slightly better than both Poisson and NB for background bins.

## Related tools

- **HPeak** (Partitions the genome into 25 bp bins, counts HDFs per bin, and classifies bins as ChIP-enriched or background via a two-state HMM with GP/ZIP emissions, providing the count data and region classes to fit) — https://github.com/cauyrd/HPeak
- **scipy.stats** (Python library used to fit the GP, ZIP, Poisson, and negative binomial distributions to HDF counts by maximum likelihood)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPREFIX -fmin 100 -fmax 300 -r 36 -interfiles
```

## Evaluation signals

- For ChIP-enriched bins, GP achieves the best AIC/log-likelihood: much better than Poisson and slightly better than negative binomial, matching the reported ranking
- For background bins, ZIP achieves a slightly better fit than both Poisson and NB, matching the reported (though untabulated) result
- Poisson is clearly the worst fit for enriched-bin HDF counts, confirming overdispersion beyond unit variance-to-mean
- Ranking table (distribution_fit_comparison.csv) reproduces the same ordering across both NRSF and STAT1 datasets
- MLE parameter estimates are finite and the fitted distributions' variance exceeds the mean for enriched bins (consistent with GP/NB rather than Poisson)

## Limitations

- The background-region ZIP fit result was reported as 'data not shown' in the article, so the ZIP-vs-Poisson/NB comparison for background bins lacks a published table to validate against
- The fits are tied to HDF counts from directional read extension with HPeak's unbalanced down-weighting at fragment ends; other fragment models may yield different count distributions
- Performance conclusions should be interpreted with caution because true positive/negative loci for experimental ChIP-Seq data are largely unknown

## Evidence

- [discussion] GP best for enriched bins over Poisson and NB: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] ZIP slightly better than Poisson and NB for background: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [discussion] Genome partitioned into 25 bp bins with HDF counts: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] Two-state HMM with GP/ZIP emissions: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] Directional read extension to form HDFs: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
