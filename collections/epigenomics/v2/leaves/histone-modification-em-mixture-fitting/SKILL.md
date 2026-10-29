---
name: histone-modification-em-mixture-fitting
description: Use when comparing ChIP-seq profiles of broad histone modifications (e.g.
  H3K27me3, H3K9me3) between two samples to call modified or differentially modified
  regions, and peak-style callers would misclassify diffuse signals.
when_to_use_negative:
- The target histone mark produces sharp peak-like features rather than broad genomic
  footprints — methods designed for peak-like features are the article's stated motivation,
  and vice versa for narrow marks this HMM is not the intended tool
- Input is already a feature/bin count matrix rather than aligned BAM reads — binning
  and count aggregation steps would be redundant and should be skipped
- The analysis requires more than two ChIP-seq samples compared simultaneously — the
  bivariate HMM is defined for a two-sample comparison
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: histoneHMM
  role: Bivariate HMM implemented in C++/R that performs EM mixture fitting of emissions,
    Baum-Welch transition estimation, and forward-backward posterior calculation on
    binned bivariate read counts
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM runs and integrates with Bioconductor
    tool sets
- name: Bioconductor
  role: Ecosystem supplying countBamInGRanges/getBins used to bin the genome and count
    reads per bin before HMM fitting
- name: samtools
  role: Removes duplicated reads from BAM files prior to binning and HMM fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:a59f5e1f6a229e7f727b9cb4a47abc327e6b2dc54048c34ce576c053fc2c176e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# histone-modification-em-mixture-fitting

## Summary

Fit a bivariate Hidden Markov Model to binned ChIP-seq read counts from two samples using histoneHMM, estimating emission distributions via EM, transition probabilities via Baum-Welch, and per-bin posterior modification probabilities via the forward-backward algorithm, to classify genomic bins as modified in both samples, unmodified in both, or differentially modified.

## When to use

Use when comparing ChIP-seq profiles of broad histone modifications (e.g. H3K27me3, H3K9me3) between two samples to call modified or differentially modified regions, and peak-style callers would misclassify diffuse signals.

## When NOT to use

- The target histone mark produces sharp peak-like features rather than broad genomic footprints — methods designed for peak-like features are the article's stated motivation, and vice versa for narrow marks this HMM is not the intended tool
- Input is already a feature/bin count matrix rather than aligned BAM reads — binning and count aggregation steps would be redundant and should be skipped
- The analysis requires more than two ChIP-seq samples compared simultaneously — the bivariate HMM is defined for a two-sample comparison

## Inputs

- Duplicated-read-removed BAM files for two ChIP-seq samples (e.g. BN.bam and SHR.bam with .bai indexes)
- Chromosome-length table (chroms.txt) for the reference genome assembly (e.g. rn4)
- Optional gene annotation GFF (ensembl59-genes.gff) and RNA-seq expression table for preprocessing

## Outputs

- Per-bin probabilistic state classifications (modified in both, unmodified in both, differentially modified)
- Differentially modified regions with genomic coordinates and posterior probabilities (e.g. differential_regions_summary.csv)
- Aggregated summary statistics such as total differential span in Mb and percent of genome

## How to apply

Partition the genome (e.g. rat rn4) into equally sized 1000 bp bins using getBins and aggregate bivariate per-bin read counts for the two samples with countBamInGRanges via preprocess.for.hmm, after removing duplicate reads with samtools and truncating read counts at the upper 0.1 percentile to avoid artifacts from extreme counts. Fit the bivariate HMM with run.bivariate.hmm: marginal emission distributions are estimated as log-normal mixtures with the EM algorithm, transition probabilities between the hidden states are learned with the Baum-Welch algorithm, and per-bin posterior enrichment probabilities are computed with the forward-backward algorithm. The model classifies each bin probabilistically as modified in both samples, unmodified in both, or differentially modified, requiring no further tuning parameters. Call differentially modified regions with get.differential.regions at the chosen cutoff and summarize genomic spans (e.g. total differential span in Mb and percent of genome).

## Related tools

- **histoneHMM** (Bivariate HMM implemented in C++/R that performs EM mixture fitting of emissions, Baum-Welch transition estimation, and forward-backward posterior calculation on binned bivariate read counts) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM runs and integrates with Bioconductor tool sets)
- **Bioconductor** (Ecosystem supplying countBamInGRanges/getBins used to bin the genome and count reads per bin before HMM fitting)
- **samtools** (Removes duplicated reads from BAM files prior to binning and HMM fitting)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- EM convergence: the emission mixture parameter estimation terminates within max.it iterations at the eps tolerance (hmm.baumwelch arguments meanlog, sdlog, max.it, eps, rho)
- Posterior sanity check: hmm.posterior output yields per-bin enrichment probabilities in [0,1] that sum consistently across hidden states via the forward-backward algorithm
- Reproduction benchmark: applying the pipeline to SHR vs BN rat heart H3K27me3 data (1000 bp bins, upper 0.1 percentile truncation) should recover ~24.96 Mb (0.9% of the rn4 genome) as differentially modified, per Table 2
- Output schema check: differential_regions_summary.csv exists with per-region coordinates and state classifications
- State decomposition: bins classified as modified-in-both, unmodified-in-both, and differentially-modified should collectively cover the genome without overlap

## Limitations

- Extreme read counts can cause artifacts and numerical problems with very small emission probabilities; read counts must be truncated at the upper 0.1 percentile
- The package version history notes dependency issues: version 1.6 removed the GNU scientific library dependency and 1.7 was needed for Rcpp > 0.12 compatibility
- The paper's read counts were generated after aligning SHR samples against a BN reference genome with known SNP positions substituted by SHR alleles; omitting allele-aware alignment may bias calls
- No version number or archival (Zenodo) DOI for the software is stated; availability is via a project website URL

## Evidence

- [abstract] Bivariate read counts drive unsupervised classification: "histoneHMM aggregates short-reads over larger regions and takes the resulting bivariate read counts as inputs for an unsupervised classification procedure"
- [discussion] Genome binned into 1000 bp bins: "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Baum-Welch transition estimation: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [discussion] Forward-backward per-bin posteriors: "calculate the probability of enrichment for each bin in the genome using the forward-backward algorithm [45]"
- [discussion] Read count truncation at 0.1 percentile: "we truncated read counts at the upper 0.1 percentile"
- [discussion] EM parameter estimation: "Parameter estimates are obtained via the EM algorithm [43]"
- [methods] Baum-Welch function signature in the package: "hmm.baumwelch(x, meanlog, sdlog, max.it, eps, rho)"
- [readme] Installation from GitHub: "devtools::install_github("matthiasheinig/histoneHMM")"
