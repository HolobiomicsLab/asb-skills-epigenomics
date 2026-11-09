---
name: bivariate-hidden-markov-model-fitting
description: Use when comparing ChIP-seq profiles of two samples for a broad, diffuse
  histone modification and you need genome-wide calls of differentially modified regions
  rather than peak-like enrichment calls.
when_to_use_negative:
- The histone mark produces sharp peak-like footprints rather than broad diffuse domains
  — methods designed for peak-like features were noted by the authors to behave poorly
  on broad marks, and this skill's rationale is specific to broad footprints like
  H3K27me3
- Only a single sample is available and no pairwise comparison is intended in a differential
  two-sample setting (a univariate mode exists via run.univariate.hmm, but this skill
  is the bivariate two-sample fitting)
- Input is already a feature/region table or peak set rather than aligned short-read
  data, since the model requires binned bivariate read counts as input
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: histoneHMM
  role: R/C++ package providing the bivariate HMM fitting (run.bivariate.hmm, EM and
    Baum-Welch estimation, forward-backward posteriors) and differential-region calling
  repo: https://matthiasheinig.github.io/histoneHMM/
- name: R
  role: Computing environment in which histoneHMM runs and the workflow is executed
- name: samtools
  role: Removes duplicated reads from BAM files before counting reads per bin
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:5e01eb517e8434a0de14e4da35a21926e3b15dd805f0e88013db3c17a7828c61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - C++
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# bivariate-hidden-markov-model-fitting

## Summary

Fit the histoneHMM bivariate Hidden Markov Model on binned bivariate ChIP-seq read counts from two samples to probabilistically classify 1000 bp genomic bins as modified in both samples, unmodified in both, or differentially modified. Designed for broad histone marks (e.g. H3K27me3, H3K9me3) where peak-calling methods perform poorly.

## When to use

Use when comparing ChIP-seq profiles of two samples for a broad, diffuse histone modification and you need genome-wide calls of differentially modified regions rather than peak-like enrichment calls.

## When NOT to use

- The histone mark produces sharp peak-like footprints rather than broad diffuse domains — methods designed for peak-like features were noted by the authors to behave poorly on broad marks, and this skill's rationale is specific to broad footprints like H3K27me3
- Only a single sample is available and no pairwise comparison is intended in a differential two-sample setting (a univariate mode exists via run.univariate.hmm, but this skill is the bivariate two-sample fitting)
- Input is already a feature/region table or peak set rather than aligned short-read data, since the model requires binned bivariate read counts as input

## Inputs

- Aligned ChIP-seq BAM files for one histone mark in two samples (e.g. H1-hESC and K562, hg19 assembly)
- Genome bin definition (1000 bp bins, from getBins with chromosome lengths table)
- histoneHMM bivariate read-count matrix produced by countBamInGRanges after duplicate removal and maxq truncation

## Outputs

- Per-bin posterior probabilities of the states: modified in both, unmodified in both, differentially modified
- Differentially modified regions per histone mark (GRanges/bed-style intervals)
- Total Mb of differentially modified genome per mark (e.g. 843.03 Mb H3K9me3, 284.58 Mb H3K79me2, 324.85 Mb H3K36me3, 591.27 Mb H3K27me3 in the ENCODE H1-hESC vs K562 comparison)

## How to apply

Remove duplicated reads from the aligned BAM files with samtools, then partition the genome (hg19 in the ENCODE example) into equally sized 1000 bp bins using getBins and aggregate bivariate read counts per sample pair with countBamInGRanges. Truncate read counts at the upper 0.1 percentile (maxq) to avoid artifacts from extreme counts and numerical problems with very small emission probabilities. For each histone mark, fit the bivariate HMM with run.bivariate.hmm, estimating emission parameters via the EM algorithm and state transition probabilities with the Baum-Welch algorithm, then compute per-bin posterior enrichment probabilities with the forward-backward algorithm. Finally, call differential regions with get.differential.regions and sum the lengths of differentially modified bins to report total Mb per mark.

## Related tools

- **histoneHMM** (R/C++ package providing the bivariate HMM fitting (run.bivariate.hmm, EM and Baum-Welch estimation, forward-backward posteriors) and differential-region calling) — https://matthiasheinig.github.io/histoneHMM/
- **R** (Computing environment in which histoneHMM runs and the workflow is executed)
- **samtools** (Removes duplicated reads from BAM files before counting reads per bin)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Differential-region totals per mark should fall in a biologically plausible range; in the published ENCODE H1-hESC vs K562 run histoneHMM called 9%-26% of the human genome as differentially modified
- Replication of published Table 2 values (843.03 Mb H3K9me3, 284.58 Mb H3K79me2, 324.85 Mb H3K36me3, 591.27 Mb H3K27me3) when using the same ENCODE GEO accessions and hg19
- EM and Baum-Welch estimation should converge (max.it/eps criteria satisfied) and posterior probabilities per bin should sum consistently across the three states
- Read counts after maxq truncation should show no values above the upper 0.1 percentile of the observed count distribution
- Sanity-check on the package's bundled BN/SHR test data reproduces modified/unmodified/differential classifications

## Limitations

- Read counts above the upper 0.1 percentile must be truncated to avoid numerical problems with very small emission probabilities; skipping this step can destabilize the fit
- The ENCODE GEO accession list in the article contains duplicates, making the exact sample list ambiguous for reproduction
- histoneHMM is distributed via the project website/GitHub without a stated archival version or Zenodo DOI in the article, complicating exact version pinning (README mentions version 1.7 requires Rcpp > 0.12 and version 1.6 dropped the GNU scientific library dependency)

## Evidence

- [abstract] we introduce histoneHMM, a powerful bivariate Hidden Markov Model for the differential analysis of histone modifications with broad genomic footprints: "we introduce histoneHMM, a powerful bivariate Hidden Markov Model for the differential analysis of histone modifications with broad genomic footprints"
- [discussion] Genome binning and count truncation: "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Count truncation threshold: "All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile"
- [discussion] Transition probability estimation: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [methods] Package installation: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] Package scope from README: "It allows for calling modified regions in single samples as well as for calling differentially modified regions in a comparison of two samples."
