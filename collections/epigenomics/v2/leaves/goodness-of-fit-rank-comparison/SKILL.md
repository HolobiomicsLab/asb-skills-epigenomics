---
name: goodness-of-fit-rank-comparison
description: Use when ChIP-Seq HDF counts per 25 bp bin must be assigned a probability
  model, and competing discrete distributions need to be fitted and ranked (e.g. GP
  vs. Poisson vs. NB) via AIC/log-likelihood.
when_to_use_negative:
- The count data are continuous or non-genomic-binned; the GP/ZIP/Poisson/NB comparison
  here is specific to discrete HDF bin counts.
- Bin-level enrichment classification is unavailable — the skill requires pre-partitioned
  ChIP-enriched vs. background bins from an HMM state assignment.
- The goal is peak boundary definition or motif discovery rather than distribution
  selection; use HPeak peak calling or downstream MatInspector-based motif analysis
  instead.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3168
tools:
- name: HPeak
  role: Generates the 25 bp binned HDF counts and two-state HMM (GP/ZIP emission)
    classification of ChIP-enriched vs. background bins used as input to distribution
    fitting
  repo: github.com/cauyrd/HPeak
- name: scipy.stats
  role: Maximum-likelihood fitting of GP, Poisson, NB, and ZIP distributions to HDF
    count vectors and computation of log-likelihood/AIC for ranking
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:8c052f7f7323738c6c0309a80647a2cb88b23c69a91a94be027bbec1719111b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HPeak
  - scipy.stats
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# goodness-of-fit-rank-comparison

## Summary

Fits candidate count distributions (GP, ZIP, Poisson, negative binomial) by maximum likelihood to HDF bin counts from HPeak ChIP-Seq analysis and ranks them by a goodness-of-fit criterion to select the best emission model for enriched vs. background genomic bins.

## When to use

Use when ChIP-Seq HDF counts per 25 bp bin must be assigned a probability model, and competing discrete distributions need to be fitted and ranked (e.g. GP vs. Poisson vs. NB) via AIC/log-likelihood.

## When NOT to use

- The count data are continuous or non-genomic-binned; the GP/ZIP/Poisson/NB comparison here is specific to discrete HDF bin counts.
- Bin-level enrichment classification is unavailable — the skill requires pre-partitioned ChIP-enriched vs. background bins from an HMM state assignment.
- The goal is peak boundary definition or motif discovery rather than distribution selection; use HPeak peak calling or downstream MatInspector-based motif analysis instead.

## Inputs

- HDF counts per 25 bp genomic bin from NRSF and STAT1 ChIP-Seq data (HPeak output)
- HMM-based bin classification (ChIP-enriched vs. background) from HPeak two-state HMM

## Outputs

- Ranked distribution-fit comparison table (distribution_fit_comparison.csv) with AIC/log-likelihood per candidate distribution per region type
- Maximum-likelihood parameter estimates for GP, ZIP, Poisson, and NB fits

## How to apply

Partition the genome into 25 bp bins and count hypothetical DNA fragments (HDFs) per bin using HPeak (NRSF and STAT1 ChIP-Seq data), then split bins into ChIP-enriched and background classes according to the HPeak two-state HMM classification. Fit the generalized Poisson (GP), Poisson, and negative binomial (NB) distributions to enriched-bin HDF counts, and the zero-inflated Poisson (ZIP), Poisson, and NB to background-bin counts, estimating parameters by maximum likelihood in Python (scipy.stats). Compare fitted distributions per dataset and region type using a goodness-of-fit criterion such as AIC or log-likelihood, and rank the candidates. Correct application reproduces the reported ranking: GP best for enriched bins (much better than Poisson, slightly better than NB) and ZIP slightly better than both Poisson and NB for background bins.

## Related tools

- **HPeak** (Generates the 25 bp binned HDF counts and two-state HMM (GP/ZIP emission) classification of ChIP-enriched vs. background bins used as input to distribution fitting) — github.com/cauyrd/HPeak
- **scipy.stats** (Maximum-likelihood fitting of GP, Poisson, NB, and ZIP distributions to HDF count vectors and computation of log-likelihood/AIC for ranking)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t NRSF.bed -c control.bed -n NRSF_out -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- The fit ranking matches the article's report: GP best for ChIP-enriched bins, much better than Poisson and slightly better than NB.
- For background bins, ZIP slightly outperforms both Poisson and NB.
- Distribution fits are performed separately for each dataset (NRSF, STAT1) and each region type (enriched vs. background), not pooled.
- Ranking table written to distribution_fit_comparison.csv contains one ranked entry per candidate distribution per region type with an AIC or log-likelihood score.

## Limitations

- The background-region ZIP vs. Poisson/NB comparison result was reported as 'data not shown' in the article, so the expected ranking must be validated indirectly.
- Whether NB or other heavy-tailed distributions fit ChIP-Seq data better than Poisson was investigated but not exhaustively benchmarked beyond the candidate set here.
- HMM bin classification depends on HPeak's initial rough estimates from summary statistics; misclassification of bins would propagate into the distribution fits.

## Evidence

- [discussion] GP fit ranking for enriched bins: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] ZIP fit for background bins: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [discussion] binning of HDFs: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] HMM two-state classification: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [readme] README command invocation: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36"
