---
name: m5c-methylation-motif-discovery
description: Use when RIMS-seq paired-end Illumina reads of a bacterial strain need
  de novo m5C methylase motif discovery, especially when multiple methylases (e.g.
  A. calcoaceticus) or dcm motifs (E. coli K12 CCWGG) are expected.
when_to_use_negative:
- The sequencing library was prepared by tagmentation rather than adapter ligation
  — RIMS-seq does not work with tagmentation-based protocols
- Precise genomic locations of individual methylated sites are needed — use EM-seq
  or bisulfite-seq instead
- m5C detection via PacBio SMRT or Nanopore is planned — SMRT sequencing usually misses
  m5C unless enhanced with TET enzyme treatment
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: mapping.pl
  role: Maps paired-end RIMS-seq reads to the reference genome with BWA-mem (--fq1,
    --fq2, --genome, --out) producing a BAM
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Splits the BAM into Read 1 and Read 2 mpileup files using samtools mpileup
    for strand-aware motif discovery
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: get_motif_all.pl
  role: Runs MosDi on the two mpileup files to identify the methylase specificity
    motifs and significance scores
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: RIMS-seq.pl
  role: 'Option 1 wrapper: assembles the reference from RIMS-seq reads with SPAdes
    and runs the downstream mapping/splitting/motif-discovery steps'
  repo: https://github.com/Ettwiller/RIMS-seq/
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:ea670b9d68a74278031a50cb8bd4d964fb72e70673d1fbbccd4e8c9bf24ee1f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - RIMS-seq.pl
  - get_motif_all.pl
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# m5c-methylation-motif-discovery

## Summary

De novo identification of m5C methylase specificities from RIMS-seq reads by mapping with BWA-mem, splitting Read1/Read2 mpileups via samtools, and running MosDi-based motif discovery (get_motif_all.pl) to recover methylated motifs with significance scores.

## When to use

Use when RIMS-seq paired-end Illumina reads of a bacterial strain need de novo m5C methylase motif discovery, especially when multiple methylases (e.g. A. calcoaceticus) or dcm motifs (E. coli K12 CCWGG) are expected.

## When NOT to use

- The sequencing library was prepared by tagmentation rather than adapter ligation — RIMS-seq does not work with tagmentation-based protocols
- Precise genomic locations of individual methylated sites are needed — use EM-seq or bisulfite-seq instead
- m5C detection via PacBio SMRT or Nanopore is planned — SMRT sequencing usually misses m5C unless enhanced with TET enzyme treatment

## Inputs

- Paired-end Illumina RIMS-seq fastq.gz read files (adapters removed), downsampled to 1-5 million read pairs
- Reference genome FASTA (exact match to sequenced strain) or SPAdes assembly from RIMS-seq reads

## Outputs

- Methylase specificity list (e.g. methylase_specificities.txt) with motif sequences and MosDi significance P-values
- Mapped BAM file and Read1/Read2 mpileup intermediate files

## How to apply

Downsample paired-end reads to 1-5 million read pairs to keep MosDi motif discovery tractable. Map reads to a reference (or a SPAdes-assembled genome from the RIMS-seq reads when no exact reference exists) with mapping.pl using BWA-mem in paired-end mode, ensuring adapters were removed beforehand. Split the BAM with split_mapped_reads.pl into Read1 and Read2 mpileup files via samtools mpileup (defaults -Q 10 base quality, -q 0 mapping quality). Run get_motif_all.pl on the two mpileups with MosDi, setting --genome to the reference fasta, --qualityscore 35 (recommended 30-35 for Illumina), --flank 7 (or 8), and --significance 1e-50. The output motif list reports each motif with its significance; e.g. A. calcoaceticus ATCC 49823 yields CGCG (P=2.33e-174) and GATC (P=3.02e-1308), while E. coli K12 time courses yield CCWGG with methylated cytosine at position 2 (P=9.71e-77, 4.25e-858, 3.61e-4371 for 10, 60, 180 min alkaline treatment).

## Related tools

- **mapping.pl** (Maps paired-end RIMS-seq reads to the reference genome with BWA-mem (--fq1, --fq2, --genome, --out) producing a BAM) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Splits the BAM into Read 1 and Read 2 mpileup files using samtools mpileup for strand-aware motif discovery) — https://github.com/Ettwiller/RIMS-seq/
- **get_motif_all.pl** (Runs MosDi on the two mpileup files to identify the methylase specificity motifs and significance scores) — https://github.com/Ettwiller/RIMS-seq/
- **RIMS-seq.pl** (Option 1 wrapper: assembles the reference from RIMS-seq reads with SPAdes and runs the downstream mapping/splitting/motif-discovery steps) — https://github.com/Ettwiller/RIMS-seq/

## Examples

```
perl mapping.pl --fq1 reads.1.fastq.gz --fq2 reads.2.fastq.gz --genome reference.fasta --out mapped.bam; perl split_mapped_reads.pl -bam mapped.bam -genome reference.fasta -mpileup1 r1.mpileup -mpileup2 r2.mpileup; perl get_motif_all.pl --genome reference.fasta --qualityscore 35 --flank 7 --significance 1e-50
```

## Evaluation signals

- Output motif list contains the expected methylated motif(s) passing the --significance 1e-50 cutoff (e.g. CGCG P=2.33e-174 and GATC P=3.02e-1308 for A. calcoaceticus ATCC 49823, confirmed by MFRE-seq)
- E. coli K12 recovery yields CCWGG with the methylated cytosine at position 2 and P-values matching the reported 9.71e-77 (10 min), 4.25e-858 (60 min) and 3.61e-4371 (180 min)
- BAM was generated with BWA-mem in paired-end mode after adapter removal, and split_mapped_reads.pl produced two mpileup files (Read1 and Read2)
- Input reads were downsampled to 1-5 million pairs so MosDi motif discovery completes without excessive processing time
- MosDi parameters fall in recommended ranges: --qualityscore 30-35 for Illumina data and --flank 7 (default) or 8

## Limitations

- RIMS-seq requires adapter-ligation library preparation and does not work with tagmentation-based protocols
- For option 2 the reference genome must be exactly the same as the sequenced strain; otherwise use option 1 (SPAdes assembly from the RIMS-seq reads)
- Motif detection needs enough read coverage to compensate for scarcity of sites; low-coverage or rare motif sites may not yield detectable signal
- In mixed communities, uneven species representation and SNPs can increase background noise, and assigning motifs to closely related species is challenging
- Skipping the column-based cleaning step (using a small library volume for amplification) is theoretically possible but untested

## Evidence

- [methods] This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi.: "This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi."
- [methods] Downsampling recommendation: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [methods] MoSDi parameter defaults: "`--significance` : motif significance (default = 1e-50)"
- [results] Multi-methylase finding: [evidence span withheld — anchor status: not_found]
- [readme] Adapter ligation requirement: "Note that the sequencing library should be done using a protocol based on adapter ligation."
- [readme] Tagmentation incompatibility: "RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies"
