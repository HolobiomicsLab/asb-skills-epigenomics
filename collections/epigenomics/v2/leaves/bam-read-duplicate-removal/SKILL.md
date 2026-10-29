---
name: bam-read-duplicate-removal
description: Use when raw ChIP-seq BAM alignments (e.g. SHR/BN rat H3K27me3 or CD-1
  mouse H3K9me3 data) are about to be aggregated into fixed-width bins for downstream
  read-count-based modeling, and duplicates have not yet been removed.
when_to_use_negative:
- Input is already a preprocessed binned read-count table (e.g. BN.txt / SHR.txt from
  the histoneHMM test data), where deduplication was already applied upstream
- Analysis uses single-cell or UMIs-tagged data where duplicate marking/removal strategies
  differ from bulk ChIP-seq duplicate handling
- Input is not an alignment file (e.g. BED peaks or gene expression tables), where
  read-level deduplication does not apply
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: samtools
  role: Used to remove duplicated reads from ChIP-seq BAM alignments before genome
    binning in the histoneHMM pipeline
- name: histoneHMM
  role: Downstream R/Bioconductor package whose binned read-count inputs (getBins,
    countBamInGRanges) rely on the deduplicated BAM files
  repo: github.com/matthiasheinig/histoneHMM
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:ba7b64594ac576ed10d9d5cf6e50c7b1b1df88a63bccc99be3fe99c87e3c2bc8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bam-read-duplicate-removal

## Summary

A ChIP-seq preprocessing step from the histoneHMM pipeline that removes PCR-duplicated reads from BAM alignments with samtools before genome binning, so that binned read counts fed to the bivariate HMM are not inflated by amplification artifacts.

## When to use

Use when raw ChIP-seq BAM alignments (e.g. SHR/BN rat H3K27me3 or CD-1 mouse H3K9me3 data) are about to be aggregated into fixed-width bins for downstream read-count-based modeling, and duplicates have not yet been removed.

## When NOT to use

- Input is already a preprocessed binned read-count table (e.g. BN.txt / SHR.txt from the histoneHMM test data), where deduplication was already applied upstream
- Analysis uses single-cell or UMIs-tagged data where duplicate marking/removal strategies differ from bulk ChIP-seq duplicate handling
- Input is not an alignment file (e.g. BED peaks or gene expression tables), where read-level deduplication does not apply

## Inputs

- ChIP-seq BAM alignment file per sample/replicate (e.g. BN.bam, SHR.bam)
- BAM index file (.bam.bai) for each alignment
- samtools installed and available on PATH

## Outputs

- Deduplicated BAM file per sample with duplicate reads removed
- Deduplicated alignments ready for binned read counting via countBamInGRanges

## How to apply

Apply samtools duplicate removal to each ChIP-seq alignment (BAM) file before any further processing. In the histoneHMM workflow this is the first filter step: duplicated reads are discarded prior to partitioning the genome (e.g. rat rn4 or mouse genome) into equally sized 1000 bp bins with getBins and counting reads per bin per sample with countBamInGRanges (min.mapq filtering). Removing duplicates matters because downstream bin-level read counts are truncated only at the upper 0.1 percentile, so unremoved duplicate stacks could create artificially extreme counts that distort the emission distribution estimation in run.bivariate.hmm. After deduplication, proceed directly to preprocessing.for.hmm / countBamInGRanges; keep the deduplicated BAMs (with .bai indexes) as the canonical input.

## Related tools

- **samtools** (Used to remove duplicated reads from ChIP-seq BAM alignments before genome binning in the histoneHMM pipeline)
- **histoneHMM** (Downstream R/Bioconductor package whose binned read-count inputs (getBins, countBamInGRanges) rely on the deduplicated BAM files) — github.com/matthiasheinig/histoneHMM

## Examples

```
samtools rmdup BN.bam BN.dedup.bam && samtools index BN.dedup.bam
```

## Evaluation signals

- The deduplicated BAM loads successfully with its index (.bam.bai) and is accepted by countBamInGRanges with min.mapq filtering
- Post-deduplication read counts per 1000 bp bin no longer show extreme stacks above the upper 0.1 percentile threshold used for count truncation
- The HMM fitted on deduplicated, truncated counts converges in run.bivariate.hmm without numerical problems from very small emission probabilities
- Differential calling on the rat SHR/BN H3K27me3 data yields a total differential span consistent with the reported 24.96 Mb (~0.9% of rn4), indicating duplicates did not inflate bin counts

## Limitations

- Duplicate removal addresses PCR amplification artifacts but not other count artifacts; the pipeline still requires truncating read counts at the upper 0.1 percentile to avoid artifacts from regions with extreme read counts and numerical problems with very small emission probabilities
- No specific samtools version is stated in the article, so exact deduplication behavior may vary across samtools releases

## Evidence

- [discussion] Deduplication precedes binning in the published workflow: "and removed duplicated reads using samtools"
- [discussion] Read counts greater than the upper 0.1 percentile are capped: "In order to avoid artifacts from regions with extreme read counts [40] and to avoid numerical problems with very small emission probabilities we truncated read counts at the upper 0.1 percentile"
- [discussion] Genome is partitioned into 1000 bp bins by default: "we partition the genome into m equally sized bins (1000 bp by default)"
- [readme] The histoneHMM repository ships raw BAM test data with indexes: "[BN.bam](http://histonehmm.molgen.mpg.de/data/BN.bam)"
