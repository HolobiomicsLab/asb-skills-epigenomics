---
name: zero-inflated-count-modeling
description: Use when count-per-bin data (e.g., HDF counts per 25 bp genome bin) shows
  far more zeros than a Poisson predicts and you need to choose among ZIP, Poisson,
  and NB emissions for a background state.
when_to_use_negative:
- For ChIP-enriched bin counts — the article shows the GP distribution fits these
  much better than Poisson and slightly better than NB, so GP (not ZIP) is the right
  emission model there
- For data without zero inflation (observed zero frequency close to the Poisson prediction),
  where plain Poisson suffices
- For continuous coverage values rather than discrete HDF counts per bin
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Partitions the genome into 25 bp bins, counts HDFs per bin, and applies the
    two-state HMM with GP/ZIP emissions whose background-state fit is evaluated by
    this skill
  repo: github.com/cauyrd/HPeak
- name: Python scipy.stats
  role: Maximum-likelihood fitting of ZIP, Poisson, and NB distributions to background-bin
    HDF counts and AIC/log-likelihood comparison
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:0b74ec6068ae08b99bf76ad1f9db831c697c59570f5a02330cc88d5105d12253
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HPeak
  - Python scipy.stats
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# zero-inflated-count-modeling

## Summary

Fit a zero-inflated Poisson (ZIP) distribution by maximum likelihood to low-count genomic bin data (e.g., HDF counts in background bins of ChIP-Seq data) and benchmark it against Poisson and negative binomial alternatives. In HPeak, the ZIP emission models the excess of empty 25 bp bins that Poisson underfits.

## When to use

Use when count-per-bin data (e.g., HDF counts per 25 bp genome bin) shows far more zeros than a Poisson predicts and you need to choose among ZIP, Poisson, and NB emissions for a background state.

## When NOT to use

- For ChIP-enriched bin counts — the article shows the GP distribution fits these much better than Poisson and slightly better than NB, so GP (not ZIP) is the right emission model there
- For data without zero inflation (observed zero frequency close to the Poisson prediction), where plain Poisson suffices
- For continuous coverage values rather than discrete HDF counts per bin

## Inputs

- HDF counts per 25 bp genome bin (e.g., NRSF and STAT1 ChIP-Seq datasets processed by HPeak)
- Bin-level background/enriched labels from HPeak's two-state HMM classification

## Outputs

- MLE-fitted ZIP, Poisson, and NB parameters for background-bin HDF counts
- Ranking table of distributions by AIC/log-likelihood (distribution_fit_comparison.csv)

## How to apply

First partition the genome into fixed-length 25 bp bins and count hypothetical DNA fragments (HDFs) per bin (e.g., via HPeak, which extends each short read directionally into an HDF). Classify bins as ChIP-enriched or background using HPeak's two-state HMM, then fit ZIP, Poisson, and negative binomial distributions to the background-bin counts by maximum likelihood (Python scipy.stats). Rank candidates by a goodness-of-fit criterion such as AIC or log-likelihood. Expected outcome from this paper: for background bins, ZIP gives a slightly better fit than both Poisson and NB; for ChIP-enriched bins, the generalized Poisson (GP) is preferred instead, so reserve ZIP modeling for the background state of the HMM.

## Related tools

- **HPeak** (Partitions the genome into 25 bp bins, counts HDFs per bin, and applies the two-state HMM with GP/ZIP emissions whose background-state fit is evaluated by this skill) — github.com/cauyrd/HPeak
- **Python scipy.stats** (Maximum-likelihood fitting of ZIP, Poisson, and NB distributions to background-bin HDF counts and AIC/log-likelihood comparison)

## Examples

```
python fit_dists.py --counts background_bins.csv --dists ZIP,poisson,nb --criterion AIC --output distribution_fit_comparison.csv
```

## Evaluation signals

- ZIP achieves the best (or tied-best) AIC/log-likelihood among ZIP/Poisson/NB on background-bin HDF counts, but only slightly better than Poisson and NB — a large margin would be suspicious
- Ranking of fits reproduces the published result: ZIP slightly better than both Poisson and NB for background bins
- Cross-check: on ChIP-enriched bins from the same datasets, GP ranks first (much better than Poisson, slightly better than NB), confirming bins were partitioned correctly
- Ranking table written to distribution_fit_comparison.csv with per-dataset and per-region-type rows
- HPeak classification consistency: enriched vs background labels come from the same two-state HMM used for binning

## Limitations

- The article's background-region ZIP-vs-Poisson-vs-NB comparison was reported as 'data not shown', so the published supporting figures/tables are not directly available for verification
- HMM parameter estimation uses Viterbi with quick initial estimates from summary statistics, so state labels (and hence the bins fed to ZIP fitting) depend on those initial estimates

## Evidence

- [discussion] ZIP distribution fits background HDF counts slightly better than Poisson and NB: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [discussion] GP fits enriched-bin HDF counts best, not ZIP: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] Bin partitioning of 25 bp and HDF counting feed the distribution comparison: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] Two-state HMM with GP/ZIP emissions defines the enriched/background split: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] Background-region ZIP comparison result not shown in the paper: [evidence span withheld — anchor status: not_found]
