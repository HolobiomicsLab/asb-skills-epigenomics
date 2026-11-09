---
name: bisulfite-like-ct-variant-methylation-detection
description: Use when analyzing RIMS-seq paired-end FASTQ reads from a bacterial strain
  to de novo identify m5C methylase specificities (e.g. non-CpG motifs) while only
  having a reference genome or SPAdes assembly available.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or other non-adapter-ligation
  protocols — RIMS-seq does not work in that case
- Precise genomic locations of methylation are needed (per-molecule, per-site) — EM-seq
  or bisulfite-seq are more appropriate for that purpose
- No exact reference genome exists and the assembled genome route (option 1) is not
  desired; the reference must exactly match the sequenced bacterium
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: mapping.pl
  role: Maps paired-end RIMS-seq reads to the reference genome with BWA-mem before
    methylation signal extraction; adapters must be removed first
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Splits the BAM into Read 1 and Read 2 mpileup files using samtools mpileup
    so motif discovery evaluates each read orientation separately
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: get_motif_all.pl
  role: Runs MosDi on the two mpileup files to de novo identify methylase specificity
    motifs with significance scoring
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: SPAdes
  role: Optional genome assembly step when no exact reference genome is available;
    the assembled genome is used for downstream motif analysis
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:da98912b507b16c105f2f43aef6eda7165fc31ff56b81feca6d65f0a6e51777d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - SPAdes
  - get_motif_all.pl
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bisulfite-like-ct-variant-methylation-detection

## Summary

Detects methylated m5C motifs by exploiting a bisulfite-like C-to-T deamination signal introduced by RIMS-seq library chemistry, de novo identifying methylase specificities from ordinary paired-end Illumina read data without specialized methylation sequencing.

## When to use

Use when analyzing RIMS-seq paired-end FASTQ reads from a bacterial strain to de novo identify m5C methylase specificities (e.g. non-CpG motifs) while only having a reference genome or SPAdes assembly available.

## When NOT to use

- The sequencing library was prepared by tagmentation or other non-adapter-ligation protocols — RIMS-seq does not work in that case
- Precise genomic locations of methylation are needed (per-molecule, per-site) — EM-seq or bisulfite-seq are more appropriate for that purpose
- No exact reference genome exists and the assembled genome route (option 1) is not desired; the reference must exactly match the sequenced bacterium

## Inputs

- Paired-end Illumina FASTQ reads (RIMS-seq library prepared with adapter ligation, not tagmentation)
- Reference genome FASTA (exactly matching the sequenced bacterium) or a SPAdes-derived assembled genome
- Downsampled read set of 1-5 million paired reads recommended

## Outputs

- methylase_specificities.txt listing motif sequences with significance (P-value) scores
- Read 1 and Read 2 mpileup files from split_mapped_reads.pl
- BAM file of reads mapped to the reference genome

## How to apply

RIMS-seq library prep creates a C-to-T deamination signal at methylated cytosines that is detected from standard paired-end reads rather than from bisulfite conversion. Downsample reads to 1-5 million read pairs to keep motif discovery fast, then map to the reference genome with BWA-mem via mapping.pl (adapters removed, paired-end mode). Split the BAM into Read 1 and Read 2 mpileup files with split_mapped_reads.pl, which uses samtools mpileup, so the methylation signal is evaluated per sequencing read direction. Run motif discovery with get_motif_all.pl using MosDi, setting --genome to the reference fasta, --qualityscore 35, --flank 7, and --significance 1e-50; motifs passing the significance cutoff are reported as methylase specificities. Multiple motifs can be recovered simultaneously, as demonstrated in A. calcoaceticus ATCC 49823 where both CGCG (P = 2.33e-174) and GATC (P = 3.02e-1308) were identified and confirmed by MFRE-seq.

## Related tools

- **mapping.pl** (Maps paired-end RIMS-seq reads to the reference genome with BWA-mem before methylation signal extraction; adapters must be removed first) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Splits the BAM into Read 1 and Read 2 mpileup files using samtools mpileup so motif discovery evaluates each read orientation separately) — https://github.com/Ettwiller/RIMS-seq/
- **get_motif_all.pl** (Runs MosDi on the two mpileup files to de novo identify methylase specificity motifs with significance scoring) — https://github.com/Ettwiller/RIMS-seq/
- **SPAdes** (Optional genome assembly step when no exact reference genome is available; the assembled genome is used for downstream motif analysis) — https://github.com/ablab/spades

## Examples

```
perl mapping.pl --fq1 reads_1.fastq.gz --fq2 reads_2.fastq.gz --genome reference.fasta --out mapped.bam; perl split_mapped_reads.pl -bam mapped.bam -genome reference.fasta -mpileup1 r1.mpileup -mpileup2 r2.mpileup; perl get_motif_all.pl --genome reference.fasta --qualityscore 35 --flank 7 --significance 1e-50
```

## Evaluation signals

- Output motif list contains both CGCG (P = 2.33e-174) and GATC (P = 3.02e-1308) for A. calcoaceticus ATCC 49823, both passing the --significance 1e-50 cutoff
- Recovered motifs are confirmed by an independent method (e.g. MFRE-seq) in the validation workflow
- MPileup files contain both Read 1 and Read 2 signal, since reads are split by orientation before MosDi
- Read input was downsampled to the recommended 1-5 million paired-end read range so motif discovery completes in reasonable time
- QC metrics (coverage, GC content, mapping rate) remain similar to standard DNA-seq, confirming sequencing quality was not degraded

## Limitations

- RIMS-seq signals require enough read coverage to compensate for the scarcity of methylation sites; low-coverage or rare motif sites may not yield detectable signal
- SNPs in microbial communities may confound the C-to-T deamination detection, increasing background noise for motif detection
- In mixed communities, uneven species representation can cause motifs to be identified only in the most abundant species, and assigning motifs to the correct species is challenging when closely related species co-exist
- The library must be adapter-ligation based; tagmentation-based protocols are incompatible
- How pervasive methylation must be at partially methylated sites to produce a detectable signal has not been evaluated

## Evidence

- [methods] Split mapped reads into Read 1/Read 2 mpileups: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] MosDi motif discovery from mpileups: "This script takes the two mpileup files previsouly generated and identifies the methylase specificity using a program called MosDi."
- [methods] Downsampling recommendation: "it is highly recommended to downsample the number of paired-end read to 1-5 millions to avoid lengthly processing time during the motif discovery step."
- [discussion] Bisulfite alternatives for localization: "other technologies such as EM-seq or bisulfite-seq are more appropriate as they enable the precise genomic location to be obtained"
- [readme] Library prep requirement: "Note that the sequencing library should be done using a protocol based on adapter ligation."
- [readme] Reference must match exactly: "In case of option2, your reference genome needs to be **exactly** the same as the bacteria sequenced using RIMS-seq."
