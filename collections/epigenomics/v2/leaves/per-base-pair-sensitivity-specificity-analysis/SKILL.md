---
name: per-base-pair-sensitivity-specificity-analysis
description: Use when benchmarking a peak caller for broad histone marks (e.g. H3K27me3)
  against a qPCR-validated truth set and needing threshold-free ROC comparison of
  sensitivity and specificity per base pair.
when_to_use_negative:
- Peak-like (narrow) histone marks or transcription factors, since methods searching
  for peak-like features were shown to generate false positive or false negative calls
  on broad diffuse modifications like H3K27me3
- Differential (two-sample) comparison analysis rather than single-sample detection
  of modified versus unmodified regions
- No qPCR-validated or otherwise curated ground-truth regions exist, since per-base-pair
  sensitivity and specificity cannot be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Univariate HMM peak caller evaluated; bins the genome (1000 bp default), estimates
    emissions via EM and transitions via Baum-Welch, scores bins with hmm.posterior
  repo: https://github.com/matthiasheinig/histoneHMM
- name: MACS2
  role: Competing peak caller run in broad mode on the same BAM files with author-recommended
    defaults
- name: Zinba
  role: Competing peak caller (fitzinba) run with author-recommended defaults
- name: SICER
  role: Competing peak caller run with author-recommended defaults
- name: BroadPeak
  role: Competing peak caller run with author-recommended defaults
- name: Rseg
  role: Competing peak caller run with author-recommended defaults
- name: samtools
  role: Removes duplicated reads from the BAM files before binning and peak calling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:15082d44e7a7e018a3f7a2cdfb82a67f6745f076c77fd5a4cd00b300148a2926
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - BroadPeak
  - C++
  - MACS2
  - R
  - Rseg
  - SICER
  - Zinba
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# per-base-pair-sensitivity-specificity-analysis

## Summary

Benchmark workflow for evaluating histoneHMM and competing peak callers (MACS2 broad, Zinba, SICER, BroadPeak, Rseg) on a qPCR-validated H3K27me3 ChIP-seq dataset by sweeping score thresholds and building ROC curves from per-base-pair sensitivity and specificity against 197 validated regions.

## When to use

Use when benchmarking a peak caller for broad histone marks (e.g. H3K27me3) against a qPCR-validated truth set and needing threshold-free ROC comparison of sensitivity and specificity per base pair.

## When NOT to use

- Peak-like (narrow) histone marks or transcription factors, since methods searching for peak-like features were shown to generate false positive or false negative calls on broad diffuse modifications like H3K27me3
- Differential (two-sample) comparison analysis rather than single-sample detection of modified versus unmodified regions
- No qPCR-validated or otherwise curated ground-truth regions exist, since per-base-pair sensitivity and specificity cannot be computed

## Inputs

- H3K27me3 ChIP-seq BAM files for GEO GSM721294 and GSM721306
- 197 qPCR-validated H3K27me3 regions as ground truth
- SAM/BAM alignment files for competing peak callers (MACS2, Zinba, SICER, BroadPeak, Rseg)

## Outputs

- ROC curves comparing histoneHMM versus competing callers
- Per-base-pair sensitivity and specificity values at swept score thresholds
- Reproduced Figure 5a comparison plot (e.g. figure_5a_roc_comparison.png)

## How to apply

Download the MB H3K27me3 ChIP-seq BAM files for GEO accessions GSM721294 and GSM721306 and remove duplicated reads with samtools. Partition the genome into 1000 bp bins with getBins, count reads with countBamInGRanges, and truncate read counts at the upper 0.1 percentile as preprocessing for the HMM. Fit the univariate HMM on the binned read counts with run.univariate.hmm (EM-based mixture estimation plus Baum-Welch transition learning) and score bins with hmm.posterior. Run MACS2 in broad mode, Zinba, SICER, BroadPeak, and Rseg on the same BAM files using the default parameters and thresholds recommended by each tool's authors. Sweep score thresholds for every method and compute per-base-pair sensitivity and specificity against the 197 qPCR-validated H3K27me3 regions to build ROC curves, then reproduce Figure 5a.

## Related tools

- **histoneHMM** (Univariate HMM peak caller evaluated; bins the genome (1000 bp default), estimates emissions via EM and transitions via Baum-Welch, scores bins with hmm.posterior) — https://github.com/matthiasheinig/histoneHMM
- **MACS2** (Competing peak caller run in broad mode on the same BAM files with author-recommended defaults)
- **Zinba** (Competing peak caller (fitzinba) run with author-recommended defaults)
- **SICER** (Competing peak caller run with author-recommended defaults)
- **BroadPeak** (Competing peak caller run with author-recommended defaults)
- **Rseg** (Competing peak caller run with author-recommended defaults)
- **samtools** (Removes duplicated reads from the BAM files before binning and peak calling)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); bins <- getBins(genome, chr, bin.size=1000); counts <- countBamInGRanges(bam.file, bins); hmm <- run.univariate.hmm(counts); p <- hmm.posterior(hmm)
```

## Evaluation signals

- histoneHMM's ROC curve dominates the competing callers, i.e. higher per-base-pair sensitivity at matched specificity, consistent with published Figure 5a
- Read counts are truncated at the upper 0.1 percentile and the genome is partitioned into 1000 bp bins before HMM scoring
- Competing tools (MACS2 broad, Zinba, SICER, BroadPeak, Rseg) are run with the default parameters and thresholds recommended by each tool's authors
- Sensitivity and specificity are computed against the 197 qPCR-validated H3K27me3 regions over swept thresholds covering all methods

## Limitations

- Competing tools lack stated version numbers; comparison uses default parameters and thresholds as recommended by the authors, which may not be optimal for every caller
- Benchmark is limited to one qPCR-validated H3K27me3 dataset (197 regions, GSM721294/GSM721306), so results may not generalize to other marks or datasets
- Article claims no version number or archival DOI for the histoneHMM software itself

## Evidence

- [results] histoneHMM outperformed the competing algorithms in detecting modified versus unmodified regions, as shown by the ROC curves in Figure 5a.: [evidence span withheld — anchor status: not_found]
- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [discussion] removed duplicated reads using samtools: "and removed duplicated reads using samtools"
- [discussion] We mostly used the default parameters and thresholds as recommended by the authors: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [methods] getBins(genome, chr, n, bin.size, offset): "getBins(genome, chr, n, bin.size, offset)"
- [readme] histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3: "histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3."
