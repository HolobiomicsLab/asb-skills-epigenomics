---
name: bam-read-subsampling-and-quality-filtering
description: Use when ChIP-seq/DNase-seq replicate BAMs must be normalized to equal
  read depth before QC metrics (FRiP, PBC, replicate correlation) so comparisons across
  replicates are not biased by library size.
when_to_use_negative:
- Input is raw FASTQ that has not yet been aligned — mapping must precede filtering
  and subsampling
- Single-sample analysis with no replicate comparison or cross-sample QC normalization,
  where fixed-depth subsampling discards data without benefit
- Long-read or non-ChIP/DNase assays where MAPQ > 1 unique-read filtering and 4M-read
  depth assumptions from ChiLin do not apply
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0080
tools:
- name: ChiLin
  role: Pipeline orchestrating the mapping, unique-read filtering, and subsample-to-4M-reads
    steps of the ChIP QC workflow
  repo: https://github.com/cfce/chilin
- name: BWA
  role: Default aligner producing the BAM files that are subsequently filtered and
    subsampled
  repo: https://github.com/lh3/bwa
- name: samtools
  role: Retains unique reads defined by mapping quality above 1 and subsamples BAMs
    to 4,000,000 reads
- name: seqtk
  role: Lightweight sequence toolkit installed in the ChiLin environment for FASTQ/FASTA
    processing supporting the QC pipeline
  repo: https://github.com/lh3/seqtk
- name: MACS2
  role: Consumes the subsampled BAMs for fragment-size estimation (predictd) and peak
    calling in the QC workflow
  repo: https://github.com/taoliu/MACS
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:9d546ace84d832790a574f2b5a7780526547dc5b31ff897225547064feb0b4bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - BWA
  - ChiLin
  - MACS2
  - bedClip
  - bedGraphToBigWig
  - bedtools
  - samtools
  - seqtk
  - wigCorrelate
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# BAM read subsampling and quality filtering

## Summary

Normalizes ChIP-seq BAM files by retaining uniquely mapped reads (mapping quality above 1) with samtools and subsampling each replicate to a fixed depth (4,000,000 reads) so downstream QC metrics such as MACS2 fragment-size estimation, PBC, FRiP, and wigCorrelate replicate correlation are computed on comparable input.

## When to use

Use when ChIP-seq/DNase-seq replicate BAMs must be normalized to equal read depth before QC metrics (FRiP, PBC, replicate correlation) so comparisons across replicates are not biased by library size.

## When NOT to use

- Input is raw FASTQ that has not yet been aligned — mapping must precede filtering and subsampling
- Single-sample analysis with no replicate comparison or cross-sample QC normalization, where fixed-depth subsampling discards data without benefit
- Long-read or non-ChIP/DNase assays where MAPQ > 1 unique-read filtering and 4M-read depth assumptions from ChiLin do not apply

## Inputs

- FASTQ reads (e.g. single-end ChIP-seq replicate FASTQ such as foxa1_t1.fastq)
- Reference genome index for BWA (e.g. hg19)
- Unfiltered aligned BAM files from BWA mapping
- Chromosome length file for the target genome build

## Outputs

- Uniquely-mapped filtered BAM files (MAPQ > 1)
- Subsampled BAM files at fixed depth (id_*_4000000.bam, 4,000,000 reads per replicate)
- Read counts / QC statistics feeding downstream PBC, FRiP, and replicate-correlation steps

## How to apply

Map raw reads with BWA (ChiLin's default aligner), then filter the resulting BAM with samtools to keep only unique reads defined by mapping quality above 1. Subsample every replicate BAM to exactly 4,000,000 reads (producing id_*_4000000.bam) so that fragment-size estimation, PBC evaluation, FRiP calculation, and wigCorrelate-based replicate-consistency checks operate on equal-depth input. Subsampling is done once and shared: the article notes that the PBC and meta-region read-ratio steps can be skipped if the MACS2 fragment-size step already produced the subsampled BAMs. The equal-depth BAMs are the inputs later intersected with MACS2 peak files (bedtools) for FRiP and converted to bigWigs for Pearson correlation across the genome.

## Related tools

- **ChiLin** (Pipeline orchestrating the mapping, unique-read filtering, and subsample-to-4M-reads steps of the ChIP QC workflow) — https://github.com/cfce/chilin
- **BWA** (Default aligner producing the BAM files that are subsequently filtered and subsampled) — https://github.com/lh3/bwa
- **samtools** (Retains unique reads defined by mapping quality above 1 and subsamples BAMs to 4,000,000 reads)
- **seqtk** (Lightweight sequence toolkit installed in the ChiLin environment for FASTQ/FASTA processing supporting the QC pipeline) — https://github.com/lh3/seqtk
- **MACS2** (Consumes the subsampled BAMs for fragment-size estimation (predictd) and peak calling in the QC workflow) — https://github.com/taoliu/MACS

## Examples

```
conda create -n chilin_env python=2 && conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 macs2=2.1.0 && chilin -h
```

## Evaluation signals

- Each subsampled BAM contains exactly 4,000,000 reads (files named id_*_4000000.bam)
- Filtered BAM contains only alignments with mapping quality above 1 (uniquely mapped reads)
- Downstream FRiP intersection (bedtools between peak files and 4M-read BAMs) and wigCorrelate Pearson correlation run without missing-input errors, confirming subsampled BAMs exist for every replicate
- PBC and meta-region read-ratio steps can be skipped without failure because step 3 already produced the subsampled BAMs (as documented in ChiLin's step ordering)

## Limitations

- ChiLin pins specific tool versions (bwa=0.7.13, samtools=0.1.19, bedtools=2.17.0, macs2=2.1.0, python 2 conda environment), so reproducing the exact filtering/subsampling behavior may require this legacy environment
- Fixed 4,000,000-read subsampling discards excess reads; if a library has fewer than 4M reads after filtering, the equal-depth normalization assumption breaks down
- The article's discussion section provides no concrete commands, environment specification, or output artifacts for reproducing results, so exact parameter provenance relies on the repository documentation

## Evidence

- [methods] Map reads with BWA, retain unique reads defined by mapping quality above 1 using samtools, and subsample each replicate BAM to 4,000,000 reads: [evidence span withheld — anchor status: not_found]
- [methods] step 3: sub-sample bam files and do macs2 fragment size estimation: "step 3: sub-sample bam files and do macs2 fragment size estimation."
- [methods] step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation: "step 4: sub-sample bam files(if step 3 is run, skip) and do PBC evaluation"
- [readme] conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0: "conda install bwa=0.7.13 samtools=0.1.19 bedtools=2.17.0 seqtk ucsc-bedclip ucsc-bedgraphtobigwig ucsc-wigcorrelate ucsc-wigtobigwig fastqc numpy macs2=2.1.0 bioconductor-seqlogo"
- [discussion] A user can reproduce the quality control and analysis result with only a simple command in a single pipeline process.: "A user can reproduce the quality control and analysis result with only a simple command in a single pipeline process."
