---
name: paired-end-read-alignment-to-reference
description: Use when paired-end RIMS-seq FASTQ reads must be aligned to a reference
  genome — either user-supplied (option 2) or de novo assembled with SPAdes (option
  1) — as a prerequisite for methylase specificity discovery.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or another non-adapter-ligation
  protocol — RIMS-seq does not work in that case
- No exact reference genome is available and you plan to use option 2 — the README
  states the reference needs to be exactly the same bacterium; use option 1 (SPAdes
  assembly) instead
- The analysis goal is long-read methylation calling (e.g. PacBio SMRT or Nanopore)
  rather than Illumina paired-end short-read alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: mapping.pl
  role: RIMS-seq wrapper that runs BWA-mem in paired-end mode to map reads to the
    reference and produce the .bam output
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: BWA
  role: 'Alignment engine: BWA-mem performs the paired-end read alignment against
    the indexed reference genome'
  repo: https://github.com/lh3/bwa
- name: SPAdes
  role: Assembles the reference genome from the RIMS-seq paired-end reads (option
    1) so the mapping target is derived from the same data
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:8dbd8bb38774645fc020d0965196f5cd77b2744eeff3de9b50080ed56f8b3e79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - RIMS-seq.pl
  - SPAdes
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# paired-end-read-alignment-to-reference

## Summary

Maps paired-end Illumina RIMS-seq reads to a reference (typically the SPAdes-assembled genome from the same reads) using BWA-mem via the RIMS-seq mapping.pl script, producing a .bam file that feeds the Read1/Read2 mpileup split for C-to-T methylated motif extraction.

## When to use

Use when paired-end RIMS-seq FASTQ reads must be aligned to a reference genome — either user-supplied (option 2) or de novo assembled with SPAdes (option 1) — as a prerequisite for methylase specificity discovery.

## When NOT to use

- The sequencing library was prepared by tagmentation or another non-adapter-ligation protocol — RIMS-seq does not work in that case
- No exact reference genome is available and you plan to use option 2 — the README states the reference needs to be exactly the same bacterium; use option 1 (SPAdes assembly) instead
- The analysis goal is long-read methylation calling (e.g. PacBio SMRT or Nanopore) rather than Illumina paired-end short-read alignment

## Inputs

- Paired-end FASTQ.gz RIMS-seq reads (e.g. ecoli_rims_3h_5M.1.fastq.gz / ecoli_rims_3h_5M.2.fastq.gz), adapter-removed
- Reference genome FASTA (either SPAdes assembly from the same reads, or an exactly matching external reference for option 2)
- BWA index of the reference genome

## Outputs

- .bam alignment file of paired-end reads mapped to the reference genome
- Downstream Read1 and Read2 mpileup files (produced from the .bam by split_mapped_reads.pl via samtools mpileup)

## How to apply

Assemble the reference genome from the paired-end RIMS-seq reads with SPAdes (RIMS-seq.pl option 1) so the reference is derived from the same data, or supply an exactly matching reference genome for option 2. Remove adapters from the reads prior to mapping, since the mapping.pl script requires adapter-free input. Run BWA-mem in paired-end mode through mapping.pl to align Read1/Read2 FASTQ files against the reference, generating a sorted .bam output. The .bam then feeds split_mapped_reads.pl, which uses samtools mpileup to separate Read1 and Read2 pileups used downstream for foreground C-to-T (and G-to-A) position extraction. Adapters must be removed before this step and the reference must be exactly the same bacterium as sequenced.

## Related tools

- **mapping.pl** (RIMS-seq wrapper that runs BWA-mem in paired-end mode to map reads to the reference and produce the .bam output) — https://github.com/Ettwiller/RIMS-seq/
- **BWA** (Alignment engine: BWA-mem performs the paired-end read alignment against the indexed reference genome) — https://github.com/lh3/bwa
- **SPAdes** (Assembles the reference genome from the RIMS-seq paired-end reads (option 1) so the mapping target is derived from the same data) — https://github.com/ablab/spades

## Examples

```
./bwa mem ref.fa read1.fq read2.fq | gzip -3 > aln-pe.sam.gz
```

## Evaluation signals

- A .bam file is produced by mapping.pl from the paired-end FASTQ inputs
- The reference used is either the SPAdes assembly of the same reads or exactly matches the sequenced bacterium (README requirement for option 2)
- Reads were adapter-trimmed before mapping, as mapping.pl explicitly requires adapter-free input
- The .bam yields non-empty Read1 and Read2 mpileup files when split by split_mapped_reads.pl, confirming successful paired-end alignment
- On the E. coli K12 MG1655 test data (PRJNA706563 / ecoli_rims_3h_5M reads), downstream motif discovery recovers the expected CCWGG methylated motif

## Limitations

- Adapters must be removed prior to mapping with mapping.pl; untrimmed reads will degrade the alignment
- For option 2, the reference genome must be exactly the same as the sequenced bacterium; if not available, the genome must be assembled from the reads (option 1)
- It is highly recommended to downsample to 1–5 million paired-end reads to avoid lengthy processing during the downstream motif discovery step
- In microbial communities, SNPs can confound C-to-T deamination detection downstream, increasing background noise for motif detection

## Evidence

- [methods] mapping.pl uses BWA-mem for paired-end alignment: "This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script."
- [methods] SPAdes assembly provides the mapping reference in option 1: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [methods] Reads are separated into Read 1 / Read 2 mpileups from the BWA bam: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [readme] Option 2 reference genome must exactly match the sequenced bacterium: "In case of option2, your reference genome needs to be **exactly** the same as the bacteria sequenced using RIMS-seq."
- [readme] BWA-mem paired-end invocation from the BWA README: [evidence span withheld — anchor status: external_doc]
