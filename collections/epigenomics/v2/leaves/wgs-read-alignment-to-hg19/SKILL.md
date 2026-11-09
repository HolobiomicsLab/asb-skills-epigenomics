---
name: wgs-read-alignment-to-hg19
description: Use when the input is raw WGS FASTQ (or unaligned BAM) from tumor samples
  and the goal is to reconstruct eccDNA/focal amplifications via the BWA–ReadDepth–AmpliconArchitect
  pipeline on hg19.
when_to_use_negative:
- Input alignments are already in hg19 BAM form — skip straight to ReadDepth copy-number
  calling (copy number >5, size >100 kb)
- Reference genome must be hg38 or another build — reindexing/re-liftover is needed
  since the eccDNAdb pipeline uses hg19 throughout
- Data are short-read DNA-seq of non-WGS type (e.g. CIRCLE-seq or CIDER-Seq), for
  which the article notes few public datasets exist and AmpliconArchitect is not the
  intended detector
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: BWA
  role: Aligns low-coverage WGS reads to the hg19 human reference genome as the first
    step of the eccDNAdb identification pipeline
- name: AmpliconArchitect
  role: Downstream consumer of the hg19 alignments; reconstructs focal amplifications
    and infers circular cycles from copy-number-called regions
  repo: github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:b772c8fbf66e05d3b57b4cacf521ca3197a8f43a3f5f0086afff714119a77daf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - BWA
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# wgs-read-alignment-to-hg19

## Summary

Aligns low-coverage cancer whole-genome sequencing reads (FASTQ/BAM from SRA/CCLE repositories) to the hg19 human reference genome with BWA, producing the sorted alignment files that downstream copy-number calling (ReadDepth) and eccDNA reconstruction (AmpliconArchitect) in the eccDNAdb pipeline require.

## When to use

Use when the input is raw WGS FASTQ (or unaligned BAM) from tumor samples and the goal is to reconstruct eccDNA/focal amplifications via the BWA–ReadDepth–AmpliconArchitect pipeline on hg19.

## When NOT to use

- Input alignments are already in hg19 BAM form — skip straight to ReadDepth copy-number calling (copy number >5, size >100 kb)
- Reference genome must be hg38 or another build — reindexing/re-liftover is needed since the eccDNAdb pipeline uses hg19 throughout
- Data are short-read DNA-seq of non-WGS type (e.g. CIRCLE-seq or CIDER-Seq), for which the article notes few public datasets exist and AmpliconArchitect is not the intended detector

## Inputs

- Low-coverage WGS FASTQ files from SRA study SRP081035
- Low-coverage WGS FASTQ/BAM files from CCLE BioProject PRJNA523380
- hg19 human reference genome FASTA (with BWA index)

## Outputs

- Per-sample BWA alignments against hg19 (SAM/BAM) in hg19 genomic coordinates
- Sorted, indexed BAM files ready for ReadDepth copy-number calling

## How to apply

Download the low-coverage WGS FASTQ/BAM files from the SRA study SRP081035 and the CCLE WGS BioProject PRJNA523380. Align the WGS reads to the hg19 human reference genome using BWA (BWA-MEM is the standard short-read mode), producing sorted BAM files with per-sample read placement on hg19 coordinates. Keep the reference, blacklist-region conventions, and coordinate system consistent with hg19 because subsequent ReadDepth copy-number calling (retaining copy number >5 and size >100 kb) and AmpliconArchitect cycle reconstruction all consume these hg19-based alignments. Alignment quality directly gates eccDNA sensitivity, so low-coverage WGS samples should be checked for sufficient mapped reads before copy-number calling.

## Related tools

- **BWA** (Aligns low-coverage WGS reads to the hg19 human reference genome as the first step of the eccDNAdb identification pipeline)
- **AmpliconArchitect** (Downstream consumer of the hg19 alignments; reconstructs focal amplifications and infers circular cycles from copy-number-called regions) — github.com/solo7773/eccDNAdb

## Examples

```
bwa mem -t 16 hg19.fa sample_R1.fastq sample_R2.fastq | samtools sort -o sample.hg19.sorted.bam; samtools index sample.hg19.sorted.bam
```

## Evaluation signals

- Sorted BAM files are indexable (samtools quickcheck / .bai present) and reads map to hg19 chromosome names
- Each input FASTQ yields exactly one alignment output with a read count consistent with the raw file
- Downstream ReadDepth step produces candidate regions with copy number >5 and size >100 kb, indicating usable coverage
- Final eccDNA catalog (wgs_ecdna_catalog.tsv) is a parseable TSV/BED containing ~754 WGS-derived records for this data scope

## Limitations

- All eccDNAs in the resulting database were identified by the AmpliconArchitect pipeline only; eccDNAs detectable by other methods (e.g. CIRCLE-seq, CIDER-Seq) may be missed
- Low-coverage WGS restricts sensitivity — eccDNA calls are limited to amplified regions (copy number >5, size >100 kb) and depend on the normal-sample blacklist
- Reference is hg19, so newly sequenced data mapped to newer genome builds would not be directly comparable

## Evidence

- [abstract] whole-genome sequencing (WGS) data of cancer samples were downloaded from public repositories: "whole-genome sequencing (WGS) data of cancer samples were downloaded from public repositories"
- [discussion] we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect: "we systematically identified eccDNAs in human cancers by analysis of WGS data using AmpliconArchitect"
- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [readme] README on the pipeline's source code: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
