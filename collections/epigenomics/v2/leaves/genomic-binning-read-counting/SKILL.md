---
name: genomic-binning-read-counting
description: Use when preparing BAM/Bed ChIP-seq alignments of broad histone marks
  (e.g. H3K27me3, H3K9me3) as fixed-width binned read counts before HMM-based modification
  or differential-modification calling.
when_to_use_negative:
- Analyzing sharp, peak-like histone marks or transcription-factor ChIP-seq where
  peak-calling on individual fragments is appropriate rather than broad-footprint
  bin aggregation
- Input is already a binned count table or feature-level count matrix that does not
  need genome partitioning
- Experiments requiring base-pair resolution signal tracks rather than 1000 bp fixed-width
  enrichment classification
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: R package providing getBins, countBamInGRanges and countBedInGRanges for genome
    partitioning and per-bin read counting
  repo: https://github.com/matthiasheinig/histoneHMM
- name: samtools
  role: Removes duplicated reads from BAM files before binned counting
- name: Bioconductor
  role: Infrastructure (GRanges) used by histoneHMM's binning and counting functions
provenance:
  source_task_ids:
  - task_007
  - task_002
  - task_003
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:214f180c27f1d9890c5fe820ab10040cfe6f26321f39091cf915ce83eaa4b2d8
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# genomic-binning-read-counting

## Summary

Partitions a genome into equally sized bins (1000 bp default) and aggregates ChIP-seq short-read counts per bin, producing the binned bivariate read-count table that histoneHMM uses for its Hidden Markov Model classification.

## When to use

Use when preparing BAM/Bed ChIP-seq alignments of broad histone marks (e.g. H3K27me3, H3K9me3) as fixed-width binned read counts before HMM-based modification or differential-modification calling.

## When NOT to use

- Analyzing sharp, peak-like histone marks or transcription-factor ChIP-seq where peak-calling on individual fragments is appropriate rather than broad-footprint bin aggregation
- Input is already a binned count table or feature-level count matrix that does not need genome partitioning
- Experiments requiring base-pair resolution signal tracks rather than 1000 bp fixed-width enrichment classification

## Inputs

- Duplicate-removed ChIP-seq alignment files (BAM) with index, one per sample or replicate (e.g. GSM721294/GSM721306 BAMs)
- Bed-format alignment files (alternative, via countBedInGRanges)
- Chromosome length table or genome definition for bin partitioning (e.g. chroms.txt, hg19, rn4)
- Minimum mapping quality threshold for countBamInGRanges (min.mapq)

## Outputs

- Genomic bin ranges (GRanges) of equally sized 1000 bp bins
- Per-bin read-count table with one column per sample/replicate, counts truncated at the upper 0.1 percentile
- Preprocessed count matrix ready as input to run.univariate.hmm or run.bivariate.hmm (e.g. preprocessed BN.txt/SHR.txt style files)

## How to apply

Install histoneHMM via devtools::install_github('matthiasheinig/histoneHMM'), then partition the genome into m equally sized bins (1000 bp by default) using getBins(genome, chr, bin.size) for the target assembly (e.g. hg19, mouse, rn4). Count reads per bin with countBamInGRanges(bam.file, granges, min.mapq) after removing duplicated reads with samtools; countBedInGRanges is available for Bed-format inputs. Truncate all read counts greater than the upper 0.1 percentile (maxq=0.999) to the value of the 0.1 percentile to avoid artifacts from extreme counts and numerical problems with very small emission probabilities. The resulting read-count table (one column per sample/replicate) is the direct input to run.univariate.hmm or run.bivariate.hmm.

## Related tools

- **histoneHMM** (R package providing getBins, countBamInGRanges and countBedInGRanges for genome partitioning and per-bin read counting) — https://github.com/matthiasheinig/histoneHMM
- **samtools** (Removes duplicated reads from BAM files before binned counting)
- **Bioconductor** (Infrastructure (GRanges) used by histoneHMM's binning and counting functions)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); bins <- getBins(genome, chr, bin.size=1000); counts <- countBamInGRanges("BN.bam", bins, min.mapq=20); counts[counts > quantile(counts, 0.999)] <- quantile(counts, 0.999)
```

## Evaluation signals

- Bin ranges from getBins cover the genome with uniform bin.size (default 1000 bp) and match the supplied chromosome length table
- All read-count values in the output table are <= the upper 0.1 percentile of the raw counts (truncation applied)
- Count table dimensions equal number of genomic bins x number of samples/replicates, with no missing bins
- Downstream run.univariate.hmm / run.bivariate.hmm accepts the count table without numerical errors, indicating the maxq truncation avoided extreme-count artifacts
- Re-running the pipeline on a test dataset such as BN.bam/SHR.bam yields a reproducible per-bin count matrix

## Limitations

- Truncating read counts at the upper 0.1 percentile discards information about regions with extreme read counts, trading off to avoid numerical problems with very small emission probabilities
- Bin size is a parameter setting (1000 bp default, bin.size argument) whose sensitivity/specificity was only validated via simulation studies in Additional file 1, whose contents are not included in the article
- Software is distributed via GitHub/website without version archival; workflow validated for specific assemblies (rn4, hg19, mouse) reported in the article

## Evidence

- [discussion] Partition the genome into 1000 bp bins: "we partition the genome into m equally sized bins (1000 bp by default)"
- [discussion] Truncate read counts at the upper 0.1 percentile: "we truncated read counts at the upper 0.1 percentile"
- [methods] getBins signature: "getBins(genome, chr, n, bin.size, offset)"
- [methods] countBamInGRanges signature: "countBamInGRanges(bam.file, granges, min.mapq, read.width)"
- [discussion] Duplicate removal with samtools: "and removed duplicated reads using samtools"
- [readme] Package installation from README: "install.packages("devtools")
devtools::install_github("matthiasheinig/histoneHMM")"
