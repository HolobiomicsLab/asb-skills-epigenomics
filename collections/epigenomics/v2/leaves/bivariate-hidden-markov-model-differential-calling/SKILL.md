---
name: bivariate-hidden-markov-model-differential-calling
description: Use when comparing ChIP-seq profiles of two samples (e.g. SHR vs BN rat
  heart H3K27me3) for broad, diffuse histone marks where peak-style callers produce
  false positives/negatives, and per-bin probabilistic differential calls are needed.
when_to_use_negative:
- The histone mark has sharp, peak-like footprints rather than broad diffuse domains
  — methods searching for peak-like features are the article's stated contrast, and
  narrow-peak callers are more appropriate
- Input is already a binned read-count table with no need for re-binning or duplicate
  removal, and only the HMM fitting/calling step is required
- More than two samples must be compared simultaneously — the histoneHMM bivariate
  model is designed for a two-sample comparison
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Core R/C++ package providing getBins, countBamInGRanges, run.bivariate.hmm
    (EM, Baum-Welch, forward-backward), and get.differential.regions for the bivariate
    HMM differential calling workflow
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM runs and integrates with Bioconductor
    for GRanges-based binning and counting
- name: Bioconductor
  role: Bioinformatic tool set that histoneHMM integrates with for genome interval
    handling (GRanges) during preprocessing
- name: samtools
  role: Used upstream of the HMM to remove duplicated reads from the aligned BAM files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:814584b7d24ad12d0e215e0c3be28c5e88f6481370ff5e3d6170d071954762e0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bivariate-hidden-markov-model-differential-calling

## Summary

Apply histoneHMM, a bivariate Hidden Markov Model implemented in R/C++, to call differentially modified regions between two ChIP-seq samples for broad histone marks such as H3K27me3, by aggregating short reads into 1000 bp genomic bins and classifying each bin as modified in both samples, unmodified in both, or differentially modified.

## When to use

Use when comparing ChIP-seq profiles of two samples (e.g. SHR vs BN rat heart H3K27me3) for broad, diffuse histone marks where peak-style callers produce false positives/negatives, and per-bin probabilistic differential calls are needed.

## When NOT to use

- The histone mark has sharp, peak-like footprints rather than broad diffuse domains — methods searching for peak-like features are the article's stated contrast, and narrow-peak callers are more appropriate
- Input is already a binned read-count table with no need for re-binning or duplicate removal, and only the HMM fitting/calling step is required
- More than two samples must be compared simultaneously — the histoneHMM bivariate model is designed for a two-sample comparison

## Inputs

- Duplicated-read-removed ChIP-seq BAM files for the two samples to compare (e.g. BN.bam, SHR.bam)
- Chromosome-length table for the reference genome (e.g. chroms.txt for rat rn4)
- BAM index files (.bam.bai) for the aligned reads
- Optional gene annotation in GFF format (e.g. ensembl59-genes.gff) and RNA-seq expression table for preprocessing context

## Outputs

- Differentially modified regions between the two samples with genomic coordinates (e.g. differential_regions_summary.csv)
- Probabilistic per-bin classifications as modified in both samples, unmodified in both samples, or differentially modified
- Total genomic span of differential regions (e.g. 24.96 Mb, 0.9% of the rat genome for SHR vs BN H3K27me3)

## How to apply

After removing duplicated reads with samtools, partition the reference genome (e.g. rat rn4) into equally sized 1000 bp bins with getBins and count reads per bin per sample with countBamInGRanges via preprocess.for.hmm. Truncate per-bin read counts at the upper 0.1 percentile to avoid artifacts from extreme counts and numerical problems with small emission probabilities. Fit the bivariate HMM with run.bivariate.hmm: estimate marginal emission distributions via the EM algorithm, transition probabilities between states via Baum-Welch, and per-bin posterior enrichment probabilities via the forward-backward algorithm. Call differentially modified regions between the two samples with get.differential.regions at the reported cutoff; the method is unsupervised and requires no further tuning parameters.

## Related tools

- **histoneHMM** (Core R/C++ package providing getBins, countBamInGRanges, run.bivariate.hmm (EM, Baum-Welch, forward-backward), and get.differential.regions for the bivariate HMM differential calling workflow) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM runs and integrates with Bioconductor for GRanges-based binning and counting)
- **Bioconductor** (Bioinformatic tool set that histoneHMM integrates with for genome interval handling (GRanges) during preprocessing)
- **samtools** (Used upstream of the HMM to remove duplicated reads from the aligned BAM files)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Total span of differentially modified bins in the summary table is consistent with the reported 24.96 Mb (0.9% of the rat genome) for SHR vs BN H3K27me3 from Table 2
- differential_regions_summary.csv exists with per-region coordinates and state classifications
- Per-bin read counts show no values above the upper 0.1 percentile truncation threshold
- Emission distribution parameters converged via EM and transition probabilities converged via Baum-Welch without numerical errors from small emission probabilities
- Every bin receives a posterior classification into exactly one of the three states: modified in both, unmodified in both, or differentially modified

## Limitations

- Read counts greater than the upper 0.1 percentile must be truncated, otherwise extreme regions cause artifacts and numerical problems with very small emission probabilities
- Methods designed for peak-like features generate many false positive or false negative calls on broad modifications like H3K27me3, so results depend on the broad-footprint binning design
- SHR samples were aligned against the BN reference genome with known SNP positions substituted by SHR alleles to avoid allele-bias artifacts
- The package README states no version number or archival DOI for the released software beyond GitHub installation via devtools

## Evidence

- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] truncate read counts at upper 0.1 percentile: "we truncated read counts at the upper 0.1 percentile"
- [discussion] removed duplicated reads with samtools: "and removed duplicated reads using samtools"
- [discussion] EM parameter estimation: "Parameter estimates are obtained via the EM algorithm [43]"
- [discussion] Baum-Welch transition probabilities: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [methods] install histoneHMM from GitHub: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] README test data availability: "We provide a small test data set to try out the package"
