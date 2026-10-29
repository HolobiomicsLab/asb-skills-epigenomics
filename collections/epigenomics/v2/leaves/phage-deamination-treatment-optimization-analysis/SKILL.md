---
name: phage-deamination-treatment-optimization-analysis
description: Use when paired-end RIMS-seq fastq data (e.g. Xp12 phage DNA across 0-14
  h / 0.1-0.5 M NaOH treatments) must be converted into a C-to-T deamination rate
  table or R1/R2 fold-imbalance curve to choose optimal alkaline-treatment conditions.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or a non-adapter-ligation protocol
  — RIMS-seq deamination analysis does not work with such libraries
- The goal is precise genomic localization of methylated cytosines rather than global
  treatment-optimization — use EM-seq or bisulfite-seq instead
- Input is already an assembled genome or methylation motif call set rather than paired-end
  reads aligned to a matching reference
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: mapping.pl
  role: Maps adapter-removed paired-end reads to the Xp12 reference genome using BWA-mem
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Splits the bam into Read1 and Read2 mpileup files using samtools mpileup (Q=10,
    q=0) for strand-specific C-to-T counting
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: BWA
  role: BWA-mem performs the paired-end read alignment underlying the mpileup split
  repo: https://github.com/lh3/bwa
- name: samtools
  role: Called by split_mapped_reads.pl to generate mpileup files with the specified
    quality thresholds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:36d4e55caa150b7ddd310d672632c5908503159d4a77a6cc75c9922a009b3846
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - mapping.pl
  - samtools
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# phage-deamination-treatment-optimization-analysis

## Summary

Quantifies how heat-alkaline (NaOH) treatment time and concentration drive C-to-T read-variant deamination in m5C-only phage DNA (Xp12), computing Read1/Read2 fold imbalance and deamination rates to calibrate RIMS-seq treatment conditions.

## When to use

Use when paired-end RIMS-seq fastq data (e.g. Xp12 phage DNA across 0-14 h / 0.1-0.5 M NaOH treatments) must be converted into a C-to-T deamination rate table or R1/R2 fold-imbalance curve to choose optimal alkaline-treatment conditions.

## When NOT to use

- The sequencing library was prepared by tagmentation or a non-adapter-ligation protocol — RIMS-seq deamination analysis does not work with such libraries
- The goal is precise genomic localization of methylated cytosines rather than global treatment-optimization — use EM-seq or bisulfite-seq instead
- Input is already an assembled genome or methylation motif call set rather than paired-end reads aligned to a matching reference

## Inputs

- Paired-end Illumina fastq.gz read sets per NaOH treatment condition (e.g. Xp12 RIMS-seq data from PRJNA706563)
- Xp12 reference genome FASTA
- Treatment metadata: incubation time (0-14 h) and NaOH concentration (e.g. 0.1 M vs 0.5 M)

## Outputs

- Read1 and Read2 mpileup files per condition (samtools mpileup, Q=10, q=0)
- Per-condition C-to-T read-variant counts at reference cytosines
- R1/R2 C-to-T fold imbalance per incubation time
- Deamination rate (fraction of cytosines converted to thymine) per NaOH concentration
- xp12_deamination_rates.csv with per-condition fold-imbalance and deamination-rate values

## How to apply

Download the paired-end fastq sets for each treatment condition from NCBI BioProject PRJNA706563, remove adapters, and map in paired-end mode to the Xp12 reference genome with BWA-mem via mapping.pl. Split each bam into Read1 and Read2 mpileups with split_mapped_reads.pl, which calls samtools mpileup with minimum base quality Q=10 and minimum mapping quality q=0. Count C-to-T read variants at reference cytosines separately in each Read1 and Read2 mpileup, applying a base-calling quality filter (30-35 for Illumina, --qualityscore default 30) to suppress background noise. Compute the R1/R2 C-to-T fold imbalance per incubation time (expected to rise roughly linearly, from 3.7-fold at 10 min to ~212-fold at 14 h) and the deamination rate per NaOH concentration (0.24% at 0.1 M NaOH, 3 h, up to 2.72% at 0.5 M NaOH, 3 h). Rationale: RIMS-seq exploits heat-alkaline deamination of unmethylated cytosines while m5C methylated cytosines are protected, so the R1/R2 imbalance directly reports methylase specificity; the time-course and concentration series define where the signal is strong without over-degrading sequencing quality.

## Related tools

- **mapping.pl** (Maps adapter-removed paired-end reads to the Xp12 reference genome using BWA-mem) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Splits the bam into Read1 and Read2 mpileup files using samtools mpileup (Q=10, q=0) for strand-specific C-to-T counting) — https://github.com/Ettwiller/RIMS-seq/
- **BWA** (BWA-mem performs the paired-end read alignment underlying the mpileup split) — https://github.com/lh3/bwa
- **samtools** (Called by split_mapped_reads.pl to generate mpileup files with the specified quality thresholds)

## Examples

```
./bwa mem ref.fa read1.fq read2.fq | gzip -3 > aln-pe.sam.gz
```

## Evaluation signals

- Recomputed R1/R2 C-to-T fold imbalance at 14 h matches the reported ~212-fold maximum
- Deamination rates fall within the reported 0.24% (0.1 M NaOH, 3 h) to 2.72% (0.5 M NaOH, 3 h) range
- Fold imbalance increases roughly linearly with heat-alkaline incubation time (3.7-fold at 10 min rising to 212-fold at 14 h)
- Per-condition values reproduce the curve plotted in Supplementary Figure S2C
- Output CSV xp12_deamination_rates.csv contains one fold-imbalance and deamination-rate row per treatment condition

## Limitations

- Single nucleotide polymorphisms can confound C-to-T deamination detection, increasing background noise for motif/variant detection
- RIMS-seq signals require sufficient read coverage; low-coverage or scarce motif sites may not yield detectable signal
- The analysis requires a reference genome exactly matching the sequenced organism, or de novo assembly (option 1) instead

## Evidence

- [methods] BWA-mem mapping with adapters removed: "This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script."
- [methods] Read1/Read2 split via mpileup: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] mpileup quality thresholds: "`-Q` : minimum base quality for a base to be considered (default=10)"
- [methods] base-calling quality filter: "`--qualityscore` : minimum base calling quality score (default=30)."
- [supplementary] Xp12 data accession: "The data have been deposited with links to BioProject accession number PRJNA706563 in the NCBI BioProject database"
- [discussion] treatment protocol: "only an additional 3 h sodium hydroxide incubation and an additional column-based cleaning step"
