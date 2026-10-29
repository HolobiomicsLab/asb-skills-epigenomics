---
name: bacterial-m5c-methylation-sequencing-analysis
description: Use when paired-end Illumina fastq.gz reads from heat-alkaline-treated
  bacterial DNA need m5C motif identification plus shotgun sequencing, or when C-to-T
  variant enrichment must be quantified per sequence context (e.g. CCWGG) per read
  mate.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or other non-adapter-ligation
  protocols — RIMS-seq does not work in this case
- The goal is precise genomic location of methylated cytosines; EM-seq or bisulfite-seq
  are more appropriate
- Only a non-identical reference genome exists and no SPAdes assembly can be derived
  from the data — option 2 requires the reference to be exactly the same strain
edam_operation: http://edamontology.org/operation_0329
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: mapping.pl
  role: Maps adapter-trimmed paired-end reads to the reference genome using BWA-mem,
    producing the bam that downstream mpileup splitting uses
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Splits the BWA bam into Read 1 and Read 2 mpileup files (samtools mpileup)
    with a configurable minimum base quality (-Q) for high-confidence C-to-T calls
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: get_motif_all.pl
  role: Takes the two mpileup files and identifies the methylase specificity with
    MoSDi
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: RIMS-seq.pl
  role: Pipeline wrapper that takes paired-end reads and outputs a list of methylase
    specificities (including SPAdes assembly and mapping)
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: SPAdes
  role: Genome assembly step in option 1, deriving the reference genome from the RIMS-seq
    data when no exact reference is available
  repo: https://github.com/ablab/spades
- name: BWA
  role: Provides the BWA-mem algorithm used by mapping.pl for paired-end read mapping
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:c00fb1af4b87a4a9e7028606f908cf2a1e41c28597a5a4acfb87c1d016089b78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - RIMS-seq.pl
  - SPAdes
  - get_motif_all.pl
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0329
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# bacterial-m5c-methylation-sequencing-analysis

## Summary

Analysis of bacterial m5C methylase specificity from RIMS-seq data, which deaminates methylated cytosines to appear as C-to-T read variants and simultaneously yields standard shotgun genome sequencing. It identifies methylated motifs (e.g. CCWGG) by mapping Illumina reads, splitting Read 1/Read 2 mpileups, and classifying C-to-T variants by sequence context.

## When to use

Use when paired-end Illumina fastq.gz reads from heat-alkaline-treated bacterial DNA need m5C motif identification plus shotgun sequencing, or when C-to-T variant enrichment must be quantified per sequence context (e.g. CCWGG) per read mate.

## When NOT to use

- The sequencing library was prepared by tagmentation or other non-adapter-ligation protocols — RIMS-seq does not work in this case
- The goal is precise genomic location of methylated cytosines; EM-seq or bisulfite-seq are more appropriate
- Only a non-identical reference genome exists and no SPAdes assembly can be derived from the data — option 2 requires the reference to be exactly the same strain

## Inputs

- paired-end fastq.gz reads (e.g. ecoli_rims_3h_5M.1.fastq.gz / ecoli_rims_3h_5M.2.fastq.gz, 5M read pairs)
- reference genome FASTA derived from the RIMS-seq assembly (or the exact same bacterial strain's reference genome)
- RIMS-seq library prepared by adapter ligation (not tagmentation)

## Outputs

- Read 1 and Read 2 mpileup files filtered at a minimum base quality (e.g. -Q 35)
- m5C methylase specificity / methylated motif list (via get_motif_all.pl with MoSDi)
- per-context C-to-T variant fraction table (e.g. ccwgg_ct_fractions.csv) stratified by read mate and strain

## How to apply

Downsample paired-end reads to 1–5 million read pairs, as recommended to avoid lengthy processing time during motif discovery. Map adapters-trimmed reads to a reference genome in paired-end mode with BWA-mem via mapping.pl (option 2 requires the reference genome to be exactly the same strain; otherwise use option 1 with SPAdes assembly). Split the resulting bam into Read 1 and Read 2 mpileups with split_mapped_reads.pl, setting a minimum base quality such as -Q 35 so only high-confidence C-to-T calls are retained. Either feed the mpileups to get_motif_all.pl/MoSDi for de novo motif discovery, or classify each C position by context (e.g. first C of CCWGG, W = A/T, versus other C contexts) and compute the fraction of C-to-T read variants per context per read mate per strain. For the K12 3h RIMS-seq validation case, the CCWGG C-to-T fraction should rise from ~1.9% (regular DNA-seq baseline) to ~25% in R1 of the dcm+ strain while dcm- strains show no elevation.

## Related tools

- **mapping.pl** (Maps adapter-trimmed paired-end reads to the reference genome using BWA-mem, producing the bam that downstream mpileup splitting uses) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Splits the BWA bam into Read 1 and Read 2 mpileup files (samtools mpileup) with a configurable minimum base quality (-Q) for high-confidence C-to-T calls) — https://github.com/Ettwiller/RIMS-seq/
- **get_motif_all.pl** (Takes the two mpileup files and identifies the methylase specificity with MoSDi) — https://github.com/Ettwiller/RIMS-seq/
- **RIMS-seq.pl** (Pipeline wrapper that takes paired-end reads and outputs a list of methylase specificities (including SPAdes assembly and mapping)) — https://github.com/Ettwiller/RIMS-seq/
- **SPAdes** (Genome assembly step in option 1, deriving the reference genome from the RIMS-seq data when no exact reference is available) — https://github.com/ablab/spades
- **BWA** (Provides the BWA-mem algorithm used by mapping.pl for paired-end read mapping) — https://github.com/lh3/bwa

## Examples

```
mapping.pl ecoli_rims_3h_5M.1.fastq.gz ecoli_rims_3h_5M.2.fastq.gz ref.fa && split_mapped_reads.pl ecoli_rims_3h_5M.bam -Q 35
```

## Evaluation signals

- The K12 3h RIMS-seq R1 CCWGG C-to-T fraction reaches ~25%, well above the ~1.9% regular DNA-seq baseline
- The dcm- BL21 strain shows no CCWGG-specific C-to-T elevation relative to other C contexts, confirming dcm+ specificity
- The C-to-T elevation is observed only in Read 1 (the deamination-bearing strand), not in Read 2
- Downsampled libraries are limited to 1–5 million read pairs to avoid excessively long motif discovery processing
- Sequencing QC metrics (coverage, GC content, mapping rate) of the RIMS-seq data are comparable to standard DNA-seq

## Limitations

- RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies
- Single nucleotide polymorphisms in microbial communities may confound C-to-T deamination detection, increasing background noise
- Partially methylated sites require sufficient read coverage; low-coverage or rare motif sites may not yield a detectable signal

## Evidence

- [methods] This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script.: "This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script."
- [methods] This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly.: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step.: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [methods] `-Q` : minimum base quality for a base to be considered (default=10): "`-Q` : minimum base quality for a base to be considered (default=10)"
- [readme] RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies: "RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies"
- [readme] your reference genome needs to be **exactly** the same as the bacteria sequenced using RIMS-seq: "your reference genome needs to be **exactly** the same as the bacteria sequenced using RIMS-seq"
- [discussion] single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise"
