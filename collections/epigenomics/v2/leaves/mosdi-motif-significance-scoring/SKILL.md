---
name: mosdi-motif-significance-scoring
description: Use when RIMS-seq Read 1/Read 2 mpileup files are available and the goal
  is to de novo recover m5C methylase motifs with significance scores, with or without
  a reference genome.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or other non-adapter-ligation
  protocols, since RIMS-seq does not work in that case
- The goal is precise genomic localization of methylation at individual sites rather
  than global methylase specificity — use EM-seq or bisulfite-seq instead
- The available reference genome is not exactly the same as the sequenced bacterial
  strain and no de novo assembly option is used
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0160
tools:
- name: get_motif_all.pl
  role: Takes the two Read 1/Read 2 mpileup files and identifies the methylase specificity
    by calling MosDi
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: MoSDi
  role: Motif discovery/significance scoring program invoked by get_motif_all.pl on
    the mpileup files
  repo: https://bitbucket.org/tobiasmarschall/mosdi/src/master/
- name: split_mapped_reads.pl
  role: Produces the Read 1 and Read 2 mpileup files that feed motif discovery, using
    samtools mpileup
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: RIMS-seq.pl
  role: Automates the full pipeline including assembly (Option 1) through motif discovery
  repo: https://github.com/Ettwiller/RIMS-seq/
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:c6d8d7b6d986db2b92312750972ecf8f1424a77e41e44f2d6500e87d319db1c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA-mem
  - MoSDi
  - MosDi
  - RIMS-seq.pl
  - get_motif_all.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# mosdi-motif-significance-scoring

## Summary

De novo identification of m5C methylase specificities from RIMS-seq mpileup files by running motif discovery with MosDi via the get_motif_all.pl script, scoring candidate methylated motifs (e.g. CGCG, GATC) by statistical significance.

## When to use

Use when RIMS-seq Read 1/Read 2 mpileup files are available and the goal is to de novo recover m5C methylase motifs with significance scores, with or without a reference genome.

## When NOT to use

- The sequencing library was prepared by tagmentation or other non-adapter-ligation protocols, since RIMS-seq does not work in that case
- The goal is precise genomic localization of methylation at individual sites rather than global methylase specificity — use EM-seq or bisulfite-seq instead
- The available reference genome is not exactly the same as the sequenced bacterial strain and no de novo assembly option is used

## Inputs

- Read 1 mpileup file from samtools mpileup on split mapped reads
- Read 2 mpileup file from samtools mpileup on split mapped reads
- Reference genome FASTA (exact reference genome, or SPAdes de novo assembly of the same strain)

## Outputs

- List of m5C methylase specificities with motif sequences and MosDi significance scores (e.g. methylase_specificities.txt)

## How to apply

First map the RIMS-seq paired-end reads with BWA-mem via mapping.pl (adapters removed beforehand) either to a reference genome (Option 2) or to a SPAdes de novo assembly (Option 1, as automated by RIMS-seq.pl), then split the BAM into Read 1 and Read 2 mpileup files with split_mapped_reads.pl using samtools mpileup. Run get_motif_all.pl on the two mpileup files, which calls MosDi with --genome set to the same reference fasta, --qualityscore 35, --flank 7-8, and --significance 1e-50. Downsampling paired-end reads to 1-5 million read pairs beforehand is recommended to avoid lengthy processing during the motif discovery step. The output is a list of methylase specificities with motif sequences and significance scores; e.g. A. calcoaceticus ATCC 49823 yields CGCG (P = 2.33e-174, reference mapping) and GATC (P = 3.02e-1308), or GATC (P = 1.44e-1255) and CGCG (P = 8.6e-228) against the de novo assembly.

## Related tools

- **get_motif_all.pl** (Takes the two Read 1/Read 2 mpileup files and identifies the methylase specificity by calling MosDi) — https://github.com/Ettwiller/RIMS-seq/
- **MoSDi** (Motif discovery/significance scoring program invoked by get_motif_all.pl on the mpileup files) — https://bitbucket.org/tobiasmarschall/mosdi/src/master/
- **split_mapped_reads.pl** (Produces the Read 1 and Read 2 mpileup files that feed motif discovery, using samtools mpileup) — https://github.com/Ettwiller/RIMS-seq/
- **RIMS-seq.pl** (Automates the full pipeline including assembly (Option 1) through motif discovery) — https://github.com/Ettwiller/RIMS-seq/

## Examples

```
perl get_motif_all.pl -bam mapping.bam -genome ref.fasta -mpileup1 read1.mpileup -mpileup2 read2.mpileup --qualityscore 35 --flank 7 --significance 1e-50
```

## Evaluation signals

- The output motif list contains both expected m5C motifs (e.g. CGCG and GATC for A. calcoaceticus ATCC 49823) at significance passing the --significance 1e-50 cutoff
- Significance values match reported values when reproducing the analysis (reference mapping: CGCG P = 2.33e-174, GATC P = 3.02e-1308; de novo assembly: GATC P = 1.44e-1255, CGCG P = 8.6e-228)
- Motifs recovered against a de novo SPAdes assembly match those found when mapping to the exact reference genome
- samtools mpileup splitting used default filters (-Q 10 minimum base quality, -q 0 minimum mapping quality) and MosDi ran with --qualityscore 35

## Limitations

- Downsampling to 1-5 million read pairs is highly recommended because full datasets cause lengthy processing time during motif discovery
- In Option 2 the reference genome must be exactly the same as the bacteria sequenced; otherwise use the Option 1 assembly route
- In microbial communities, closely related species confound motif-to-species assignment, SNPs can increase background noise, and RIMS-seq identifies motifs only in the most abundant species
- Enough read coverage is required; low-coverage or rare motif sites may not yield detectable signal

## Evidence

- [methods] This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi.: "This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi."
- [methods] it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step.: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [readme] In case of option2, your reference genome needs to be exactly the same as the bacteria sequenced using RIMS-seq.: "In case of option2, your reference genome needs to be **exactly** the same as the bacteria sequenced using RIMS-seq."
