---
name: em-baum-welch-hmm-parameter-estimation
description: Use when fitting a bivariate Hidden Markov Model to binned ChIP-seq read
  counts (e.g. histoneHMM on 1000 bp bins of broad marks like H3K9me3/H3K27me3) to
  classify bins as modified or differentially modified.
when_to_use_negative:
- The mark produces sharp peak-like features rather than broad diffuse footprints
  (e.g. typical transcription factor ChIP-seq), where peak-calling methods are appropriate
- Input is already a list of called regions or a feature table rather than per-bin
  bivariate read counts
- Only one sample/condition is available and no two-sample comparison is intended
  — use the univariate HMM (run.univariate.hmm) instead
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: R/C++ package providing run.bivariate.hmm, hmm.baumwelch, hmm.posterior, get.differential.regions
    for HMM fitting and differential calling
  repo: https://github.com/matthiasheinig/histoneHMM
- name: samtools
  role: Removes duplicated reads from ChIP-seq alignments before binning and read
    counting
- name: Bioconductor
  role: Ecosystem used for binning (getBins) and BAM read counting (countBamInGRanges)
    feeding the HMM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:cf0ba85e65f59cb26862fe92918f9cd169f7a25693c61eaf0a1fa6153c93390f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# em-baum-welch-hmm-parameter-estimation

## Summary

Estimate bivariate HMM parameters for histoneHMM ChIP-seq differential analysis: emission mixture parameters via the EM algorithm, state transition probabilities via Baum-Welch, and per-bin enrichment posteriors via the forward-backward algorithm on 1000 bp binned read counts.

## When to use

Use when fitting a bivariate Hidden Markov Model to binned ChIP-seq read counts (e.g. histoneHMM on 1000 bp bins of broad marks like H3K9me3/H3K27me3) to classify bins as modified or differentially modified.

## When NOT to use

- The mark produces sharp peak-like features rather than broad diffuse footprints (e.g. typical transcription factor ChIP-seq), where peak-calling methods are appropriate
- Input is already a list of called regions or a feature table rather than per-bin bivariate read counts
- Only one sample/condition is available and no two-sample comparison is intended — use the univariate HMM (run.univariate.hmm) instead

## Inputs

- BAM alignment files of ChIP-seq replicates (e.g. ENA SRP018808 CD-1 mouse liver H3K9me3) with duplicates removed via samtools
- Genome bin table from getBins (1000 bp default bin.size)
- Per-bin read-count table from countBamInGRanges with min.mapq filtering and counts truncated at the upper 0.1 percentile

## Outputs

- Fitted bivariate HMM with estimated emission mixture parameters (meanlog, sdlog) and state transition probabilities
- Per-bin posterior enrichment probabilities from the forward-backward algorithm
- Differentially modified regions from get.differential.regions (e.g. differential_regions_CD1_H3K9me3.tsv) totalling ~121.89 Mb (4.6% of mouse genome)

## How to apply

After removing duplicated reads with samtools and partitioning the genome into 1000 bp bins (getBins, bin.size=1000), count reads per bin per replicate with countBamInGRanges using min.mapq filtering, then truncate all read counts at the upper 0.1 percentile to avoid artifacts from extreme counts and numerical problems with very small emission probabilities. Fit the bivariate HMM with run.bivariate.hmm: emission mixture parameters are estimated with the EM algorithm (hmm.baumwelch takes meanlog, sdlog, max.it, eps, rho), transition probabilities between states are determined with the Baum-Welch algorithm, and per-bin enrichment probabilities are computed with the forward-backward algorithm. Call differentially modified bins with get.differential.regions using maxq=0.999, matching the 0.1 percentile truncation, then merge adjacent called bins. In the CD-1 mouse liver H3K9me3 benchmark this procedure detected 121.89 Mb (4.6% of the mouse genome) as differentially modified between males and females.

## Related tools

- **histoneHMM** (R/C++ package providing run.bivariate.hmm, hmm.baumwelch, hmm.posterior, get.differential.regions for HMM fitting and differential calling) — https://github.com/matthiasheinig/histoneHMM
- **samtools** (Removes duplicated reads from ChIP-seq alignments before binning and read counting)
- **Bioconductor** (Ecosystem used for binning (getBins) and BAM read counting (countBamInGRanges) feeding the HMM)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); fit <- run.bivariate.hmm(data=binned_counts, maxq=0.999)
```

## Evaluation signals

- Summed width of merged differential regions falls within a small tolerance of 121.89 Mb (~4.6% of the mouse genome) for the CD-1 liver H3K9me3 re-run
- All read counts used for fitting are truncated at the upper 0.1 percentile; maxq=0.999 in get.differential.regions matches this truncation
- EM/Baum-Welch convergence respects max.it and eps settings in hmm.baumwelch without numerical errors from small emission probabilities
- Called regions are genomic intervals (merged adjacent 1000 bp bins) written with proper chromosome/start/end coordinates
- Bins classified probabilistically as modified in both, unmodified in both, or differentially modified — the three-state posterior output is present

## Limitations

- Truncation of read counts at the upper 0.1 percentile caps extreme counts and may mask genuine ultra-high enrichment regions
- Supplementary parameter-setting simulations ('Additional file 1') are referenced but their contents are not included in the document
- histoneHMM is distributed via a project website/GitHub without a stated version number or archival DOI for the software itself
- Default bin size 1000 bp is assumed; different bin sizes would change the differentially modified fraction and invalidate the 121.89 Mb benchmark

## Evidence

- [discussion] Parameter estimates are obtained via the EM algorithm: "Parameter estimates are obtained via the EM algorithm [43]"
- [discussion] Baum-Welch transitions: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [discussion] Forward-backward posteriors: "calculate the probability of enrichment for each bin in the genome using the forward-backward algorithm"
- [discussion] Count truncation rationale: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [methods] hmm.baumwelch signature: "hmm.baumwelch(x, meanlog, sdlog, max.it, eps, rho)"
- [readme] Package installation: "install.packages("devtools")
devtools::install_github("matthiasheinig/histoneHMM")"
