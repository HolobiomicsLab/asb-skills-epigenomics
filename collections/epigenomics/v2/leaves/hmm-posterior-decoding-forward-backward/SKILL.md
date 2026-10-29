---
name: hmm-posterior-decoding-forward-backward
description: Use when you need probabilistic, per-genomic-region classifications of
  ChIP-seq enrichment for broad histone marks (e.g. H3K27me3, H3K9me3) rather than
  binary peak calls, in one- or two-sample comparisons.
when_to_use_negative:
- The signal consists of sharp, peak-like features (e.g. transcription factor or H3K4me3-like
  punctate marks) where peak-calling methods are appropriate and the broad-footprint
  binning model adds no benefit
- You need only a single input already reduced to a feature or region table — the
  skill expects raw aligned reads (BAM) aggregated over genomic bins
- You require fully deterministic binary state assignments without uncertainty; use
  Viterbi decoding (hmm.viterbi) instead of posterior decoding
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: 'R/C++ package implementing the bivariate HMM: run.bivariate.hmm / run.univariate.hmm
    fit the model, hmm.posterior and the forward-backward algorithm compute per-bin
    posteriors, get.differential.regions calls differential bins'
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM is compiled and run
- name: samtools
  role: Removes duplicated reads from BAM files before counting reads into bins
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:ad8c8e1b372ab9a0047957f9773e49622de43c3a20a9edf0cd8a6c49c50c77a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - C++
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# HMM posterior decoding via forward-backward algorithm

## Summary

Computes per-bin posterior probabilities of histone-modification states (modified in both samples, unmodified in both, or differentially modified) using the forward-backward algorithm within the histoneHMM bivariate Hidden Markov Model, after EM and Baum-Welch parameter estimation on binned bivariate ChIP-seq read counts.

## When to use

Use when you need probabilistic, per-genomic-region classifications of ChIP-seq enrichment for broad histone marks (e.g. H3K27me3, H3K9me3) rather than binary peak calls, in one- or two-sample comparisons.

## When NOT to use

- The signal consists of sharp, peak-like features (e.g. transcription factor or H3K4me3-like punctate marks) where peak-calling methods are appropriate and the broad-footprint binning model adds no benefit
- You need only a single input already reduced to a feature or region table — the skill expects raw aligned reads (BAM) aggregated over genomic bins
- You require fully deterministic binary state assignments without uncertainty; use Viterbi decoding (hmm.viterbi) instead of posterior decoding

## Inputs

- Aligned ChIP-seq BAM files for one sample or a sample pair (e.g. ENCODE H1-hESC and K562 BAMs on hg19), duplicate reads removed with samtools
- Genome bin partition (1000 bp bins, e.g. via getBins on the target genome assembly)
- Bivariate read count matrix per bin produced with countBamInGRanges, truncated at the upper 0.1 percentile
- Optional chromosome-length table (chroms.txt) and gene annotation (GFF) for preprocessing

## Outputs

- Per-bin posterior probabilities of enrichment / modification state for every bin in the genome
- Probabilistic classification of each bin as modified in both samples, unmodified in both, or differentially modified
- Differentially modified region sets per histone mark (e.g. from get.differential.regions), summarizable as total Mb per mark

## How to apply

First partition the genome into equally sized bins (1000 bp by default) and aggregate bivariate short-read counts per sample pair, truncating read counts at the upper 0.1 percentile (maxq) to avoid artifacts from extreme counts and numerical problems with small emission probabilities. Fit the Hidden Markov Model: estimate emission mixture parameters via the EM algorithm and transition probabilities between states via the Baum-Welch algorithm. Then run the forward-backward algorithm to calculate the probability of enrichment (posterior) for each bin in the genome. In histoneHMM these steps are exposed as run.bivariate.hmm (two-sample) or run.univariate.hmm with hmm.posterior / hmm.baumwelch functions, followed by get.differential.regions to extract differentially modified bins. The rationale is that aggregating reads over larger regions suits broad diffuse marks that peak-calling methods handle poorly, and the unsupervised procedure requires no further tuning parameters.

## Related tools

- **histoneHMM** (R/C++ package implementing the bivariate HMM: run.bivariate.hmm / run.univariate.hmm fit the model, hmm.posterior and the forward-backward algorithm compute per-bin posteriors, get.differential.regions calls differential bins) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM is compiled and run)
- **samtools** (Removes duplicated reads from BAM files before counting reads into bins)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); h <- run.bivariate.hmm(c("h1esc.bam","k562.bam"), bins=getBins(genome="hg19", bin.size=1000), maxq=0.999); diff <- get.differential.regions(h)
```

## Evaluation signals

- Posterior values are valid probabilities: all per-bin posteriors fall in [0,1] and each bin's state posteriors sum to 1
- EM/Baum-Welch convergence: max.it/eps criteria met with no numerical errors from very small emission probabilities (mitigated by maxq truncation at the upper 0.1 percentile)
- Reproduction check against published values: e.g. H1-hESC vs K562 differential totals of 843.03 Mb (H3K9me3), 284.58 Mb (H3K79me2), 324.85 Mb (H3K36me3), 591.27 Mb (H3K27me3), i.e. 9%-26% of the human genome
- Every genome bin receives a non-empty classification (modified both / unmodified both / differentially modified); no bins silently dropped
- Differential calls for repressive marks H3K27me3 and H3K9me3 should cover broad contiguous regions rather than isolated single bins

## Limitations

- Read counts greater than the upper 0.1 percentile are truncated, so extreme-count regions lose resolution above maxq
- The method targets broad genomic footprints; peak-like-feature methods applied to such data generate many false positive or false negative calls, and conversely histoneHMM is not tuned for punctate peaks
- histoneHMM is distributed without a version-pinned archival DOI in the article; versions ≥1.5/1.6 changed preprocessing and dropped the GNU scientific library dependency, so results may vary across versions
- The article lists ENCODE GEO accessions with duplicates, making the exact sample list ambiguous without checking the original publication

## Evidence

- [discussion] Calculate per-bin enrichment probabilities with forward-backward algorithm: "calculate the probability of enrichment for each bin in the genome using the forward-backward algorithm [45]"
- [discussion] Truncate read counts at upper 0.1 percentile: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [discussion] Estimate transition probabilities with Baum-Welch algorithm: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [abstract] histoneHMM outputs probabilistic classifications: "histoneHMM outputs probabilistic classifications of genomic regions as being either modified in both samples, unmodified in both samples or differentially modified between samples"
- [readme] install_github README: "devtools::install_github("matthiasheinig/histoneHMM")"
