---
name: genomic-region-span-verification
description: Use when a published pipeline reports a total differential-region span
  (e.g. Mb or % of genome) and you must reproduce the pipeline and confirm your summed
  call-set span matches that value.
when_to_use_negative:
- The modification has peak-like (narrow) footprints rather than broad marks like
  H3K27me3/H3K9me3 — histoneHMM was designed for broad diffuse modifications
- Input is already a processed peak/differential-region BED file rather than aligned
  BAM reads, so the binning-and-HMM pipeline cannot be rerun to recompute the span
- No published numeric span or percentage exists to verify against, making the comparison
  step meaningless
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Bivariate HMM that bins the genome (1000 bp), fits emissions/transition probabilities,
    and calls differentially modified regions between the two ChIP-seq samples
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which the histoneHMM package runs and in which the
    span of differential bins is summed
- name: samtools
  role: Removes duplicated reads from the BAM files before read counting
- name: Bioconductor
  role: Provides the bioinformatic infrastructure (e.g. GenomicRanges handling in
    getBins/countBamInGRanges) used by the pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:8cf70383938401081d405b024031080c3faa0bbf7aa423cfb76fc701553538c0
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# genomic-region-span-verification

## Summary

Verifies a ChIP-seq differential calling result by summing the genomic span of called differentially modified regions and comparing it against a reported total, e.g. histoneHMM's 24.96 Mb (0.9% of rat rn4) for SHR vs BN heart H3K27me3.

## When to use

Use when a published pipeline reports a total differential-region span (e.g. Mb or % of genome) and you must reproduce the pipeline and confirm your summed call-set span matches that value.

## When NOT to use

- The modification has peak-like (narrow) footprints rather than broad marks like H3K27me3/H3K9me3 — histoneHMM was designed for broad diffuse modifications
- Input is already a processed peak/differential-region BED file rather than aligned BAM reads, so the binning-and-HMM pipeline cannot be rerun to recompute the span
- No published numeric span or percentage exists to verify against, making the comparison step meaningless

## Inputs

- ChIP-seq BAM files for two conditions (e.g. BN.bam and SHR.bam rat heart H3K27me3, ArrayExpress E-MTAB-1102)
- Chromosome-length table (e.g. chroms.txt for rat rn4)
- Genome assembly (rat rn4) for bin partitioning
- Cutoff/threshold value used in the published differential calling step

## Outputs

- differential_regions_summary.csv with per-region coordinates and state classifications
- Total differentially modified span in Mb and as a percentage of the genome (expected: 24.96 Mb / 0.9% of rat genome)

## How to apply

Obtain the raw BAM files (e.g. BN.bam/SHR.bam from E-MTAB-1102 or the histoneHMM test data), remove duplicated reads with samtools, and partition the genome (rat rn4) into 1000 bp bins using getBins. Count reads per bin per sample with countBamInGRanges via preprocess.for.hmm, truncate per-bin read counts at the upper 0.1 percentile to avoid artifacts from extreme counts, and fit the bivariate HMM with run.bivariate.hmm (EM for marginal emissions, Baum-Welch for transitions, forward-backward for posteriors). Call differentially modified regions with get.differential.regions at the reported cutoff, then sum the span of bins classified as differentially modified in the summary table. Accept the reproduction if the summed span is consistent with the reported figure (24.96 Mb / 0.9% of the rn4 genome from Table 2), allowing for minor stochastic variation from EM/Baum-Welch initialization.

## Related tools

- **histoneHMM** (Bivariate HMM that bins the genome (1000 bp), fits emissions/transition probabilities, and calls differentially modified regions between the two ChIP-seq samples) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which the histoneHMM package runs and in which the span of differential bins is summed)
- **samtools** (Removes duplicated reads from the BAM files before read counting)
- **Bioconductor** (Provides the bioinformatic infrastructure (e.g. GenomicRanges handling in getBins/countBamInGRanges) used by the pipeline)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); bins <- getBins(genome, chr, bin.size=1000); pre <- preprocess.for.hmm(SHR.bam, genes, bin.size=1000, genome="rn4", chr=chroms)
```

## Evaluation signals

- Summed genomic span of bins classified as differentially modified is consistent with the reported 24.96 Mb (0.9% of the rat genome) from Table 2
- differential_regions_summary.csv exists and contains per-region genomic coordinates plus state classifications
- Read counts per bin were truncated at the upper 0.1 percentile before HMM fitting (sanity-check the preprocessed count matrix)
- Duplicate reads were removed with samtools prior to counting, matching the published preprocessing
- Bin size is 1000 bp as in the published pipeline, so spans are a multiple of bin size

## Limitations

- Version numbers for the package are not stated in the paper; the README notes later versions (1.5+ CLI, 1.6/1.7 Rcpp fixes) may change behavior
- Supplementary material 'Additional file 1' (simulation settings for parameter choices) is referenced but its contents are not included, so exact tuning details cannot be fully audited
- Extreme per-bin read counts cause numerical problems with very small emission probabilities, hence the mandatory 0.1-percentile truncation; skipping it can corrupt the HMM fit
- The paper states no changelog was found for the source repository, complicating exact version pinning for reproduction

## Evidence

- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Truncate read counts at upper 0.1 percentile: "All read counts greater than the upper 0.1 percentile were set to the value of the upper 0.1 percentile"
- [discussion] Removed duplicated reads with samtools: "and removed duplicated reads using samtools"
- [discussion] Estimate transition probabilities with Baum-Welch: "We use the Baum-Welch algorithm [44] to determine the transition probabilities between states"
- [methods] Installation via devtools from GitHub: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] README describes histoneHMM scope for broad marks and differential calling: "histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3."
