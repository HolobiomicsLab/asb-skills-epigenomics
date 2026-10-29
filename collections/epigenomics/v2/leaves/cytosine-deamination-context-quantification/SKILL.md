---
name: cytosine-deamination-context-quantification
description: Use when RIMS-seq or DNA-seq mpileup data must be classified by cytosine
  sequence context to test whether C-to-T deamination variants are enriched in a methylated
  motif (e.g. dcm CCWGG) in R1 rather than R2.
when_to_use_negative:
- Library preparation is based on tagmentation or other non-adapter-ligation technologies
  — RIMS-seq does not work in that case
- The goal is de novo motif discovery from mpileups rather than context-stratified
  quantification of known motifs — use RIMS-seq.pl / get_motif_all.pl with MoSDi instead
- Base qualities are below the -Q 35 threshold or coverage is too low to compensate
  for the scarcity of motif sites
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3673
tools:
- name: mapping.pl
  role: Maps paired-end reads to the reference genome with BWA-mem prior to context
    classification
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Splits the bam into Read 1 and Read 2 samtools mpileup files with a minimum
    base quality of 35 for stratified C-to-T counting
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: bwa
  role: BWA-mem aligner underlying mapping.pl for paired-end read mapping
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:2a52438f7dc1ce6eb0c2483f252cc7d53d469d38ee69d32be3746e409eaf6991
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - bwa
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cytosine-deamination-context-quantification

## Summary

Quantifies the fraction of high-confidence C-to-T read variants falling in the dcm CCWGG sequence context (W = A or T) versus all other C contexts, from R1/R2 mpileups of RIMS-seq or regular DNA-seq data, to detect heat-alkaline-induced C deamination signal specific to a methylated motif.

## When to use

Use when RIMS-seq or DNA-seq mpileup data must be classified by cytosine sequence context to test whether C-to-T deamination variants are enriched in a methylated motif (e.g. dcm CCWGG) in R1 rather than R2.

## When NOT to use

- Library preparation is based on tagmentation or other non-adapter-ligation technologies — RIMS-seq does not work in that case
- The goal is de novo motif discovery from mpileups rather than context-stratified quantification of known motifs — use RIMS-seq.pl / get_motif_all.pl with MoSDi instead
- Base qualities are below the -Q 35 threshold or coverage is too low to compensate for the scarcity of motif sites

## Inputs

- Paired-end fastq.gz reads downsampled to 5 million read pairs per library (e.g. ecoli_rims_3h_5M.1.fastq.gz / ecoli_rims_3h_5M.2.fastq.gz)
- Reference genome (e.g. E. coli K12 MG1655 assembly or RIMS-seq-derived assembly)
- R1 and R2 samtools mpileup files generated with -Q 35 by split_mapped_reads.pl
- Strain metadata distinguishing dcm+ (K12) and dcm- (BL21) libraries and treatment (3h heat-alkaline vs regular DNA-seq)

## Outputs

- ccwgg_ct_fractions.csv — per-read (R1/R2), per-strain, per-context (CCWGG vs other C) C-to-T variant fractions

## How to apply

Map downsampled paired-end reads (1–5 million pairs recommended to avoid lengthy processing) to the reference genome in paired-end mode with BWA-mem via mapping.pl, then split the bam into Read 1 and Read 2 mpileup files with split_mapped_reads.pl using -Q 35 so only high-confidence base calls are retained. For each mpileup, classify every C position by its local sequence context — the first C of CCWGG (W = A or T) versus all other C contexts — and count C-to-T read variants per context. Compute the fraction of C-to-T variants in CCWGG versus other contexts per read (R1, R2) and per strain (dcm+ K12, dcm- BL21) and write the table to ccwgg_ct_fractions.csv. The rationale is that heat-alkaline treatment deaminates methylated C specifically in CCWGG, and the deamination signal appears only in Read 1, so context- and read-stratified fractions isolate the methylation-dependent signal from background C-to-T noise.

## Related tools

- **mapping.pl** (Maps paired-end reads to the reference genome with BWA-mem prior to context classification) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Splits the bam into Read 1 and Read 2 samtools mpileup files with a minimum base quality of 35 for stratified C-to-T counting) — https://github.com/Ettwiller/RIMS-seq/
- **bwa** (BWA-mem aligner underlying mapping.pl for paired-end read mapping) — https://github.com/lh3/bwa

## Examples

```
mapping.pl --input ecoli_rims_3h_5M.1.fastq.gz ecoli_rims_3h_5M.2.fastq.gz --ref ref.fa && split_mapped_reads.pl --bam aln.bam -Q 35
```

## Evaluation signals

- K12 3h R1 CCWGG C-to-T fraction reaches ~25%, well above the ~1.9% DNA-seq baseline
- BL21 (dcm-) shows no elevation in CCWGG relative to other C contexts
- The CCWGG enrichment is observable only in R1, not in R2, for the dcm+ K12 strain
- Output ccwgg_ct_fractions.csv contains per-read, per-strain, per-context fraction rows with valid proportions

## Limitations

- Single nucleotide polymorphisms in microbial communities may confound the identification of the C to T deamination, increasing background noise
- RIMS-seq signals can only be obtained with enough read coverage to compensate for the scarcity of those sites; low-coverage or rare motif sites may not yield detectable signal
- Only an additional 3 h sodium hydroxide incubation plus a column-based cleaning step produces the deamination signal; skipping the cleaning step has not been tested

## Evidence

- [results] After 3 h of heat-alkaline treatment, the fraction of C-to-T read variants in the CCWGG context rises from ~1.9% in regular DNA-seq to ~25% of all C-to-T variants, an increase observable only in R1 of the dcm+ K12 strain (not in dcm- BL21).: [evidence span withheld — anchor status: not_found]
- [methods] split_mapped_reads.pl generates R1/R2 mpileups with quality filtering: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly. This is done using samtools mpileup."
- [methods] downsampling recommendation: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [methods] minimum base quality parameter: "-Q : minimum base quality for a base to be considered (default=10)"
- [readme] adapter-ligation library requirement: "Note that the sequencing library should be done using a protocol based on adapter ligation."
- [discussion] SNP confounding caveat: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise for the detection of motifs"
