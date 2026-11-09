---
name: mpileup-read-pair-splitting
description: Use when paired-end RIMS-seq or adapter-ligated Illumina reads have been
  mapped to a reference (or SPAdes-assembled) genome and separate Read1/Read2 mpileups
  are needed for m5C motif discovery.
when_to_use_negative:
- No reference or assembly exists and RIMS-seq.pl Option 1 is being run, which handles
  assembly (SPAdes), mapping, and splitting automatically
- Library preparation was tagmentation-based rather than adapter-ligation based, since
  RIMS-seq analysis is invalid for such data
- Reads are single-end, since the skill's purpose is separating Read 1 from Read 2
  of a pair
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: split_mapped_reads.pl
  role: Splits reads from the BWA-generated BAM between Read 1 and Read 2 and creates
    the two mpileup files using samtools mpileup
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: samtools
  role: Called by split_mapped_reads.pl to generate the Read1/Read2 mpileup files
    with -Q 10 and -q 0 default filters
- name: mapping.pl
  role: Upstream step that uses BWA-mem to map paired-end reads to the reference genome
    and produce the BAM input
  repo: https://github.com/Ettwiller/RIMS-seq/
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_006
  - task_007
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:cad4b40154caa9acd661d9a0c769140eb2bb2f471b03828d38f920cef8236a86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - BWA-mem
  - RIMS-seq.pl
  - mapping.pl
  - samtools
  - samtools mpileup
  - split_mapped_reads.pl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mpileup-read-pair-splitting

## Summary

Split the BAM produced by BWA-mem mapping of RIMS-seq paired-end reads into separate Read1 and Read2 samtools mpileup files, so C-to-T (and G-to-A) deamination signals can be attributed to the correct read of the pair for motif discovery.

## When to use

Use when paired-end RIMS-seq or adapter-ligated Illumina reads have been mapped to a reference (or SPAdes-assembled) genome and separate Read1/Read2 mpileups are needed for m5C motif discovery.

## When NOT to use

- No reference or assembly exists and RIMS-seq.pl Option 1 is being run, which handles assembly (SPAdes), mapping, and splitting automatically
- Library preparation was tagmentation-based rather than adapter-ligation based, since RIMS-seq analysis is invalid for such data
- Reads are single-end, since the skill's purpose is separating Read 1 from Read 2 of a pair

## Inputs

- BAM file of paired-end reads mapped with BWA-mem (from mapping.pl)
- Reference genome FASTA (exact reference for the strain, or SPAdes assembly from RIMS-seq reads)

## Outputs

- Read 1 mpileup file (samtools mpileup format)
- Read 2 mpileup file (samtools mpileup format)

## How to apply

After mapping paired-end reads with BWA-mem via mapping.pl (adapters must be removed beforehand), run split_mapped_reads.pl on the resulting BAM to separate reads into Read 1 and Read 2 and create two mpileup files using samtools mpileup. Keep the default filters -Q 10 (minimum base quality for a base to be considered) and -q 0 (minimum mapping quality for an alignment to be used), since stricter filtering is applied later at the motif-discovery step via --qualityscore (30-35 for Illumina, typically 35). The genome FASTA used for the mpileup must be the same reference used for mapping — either a provided reference identical to the sequenced strain (Option 2) or the SPAdes assembly derived from the RIMS-seq reads themselves (Option 1, automated by RIMS-seq.pl). The two output mpileup files then feed directly into get_motif_all.pl/MosDi for methylase specificity identification.

## Related tools

- **split_mapped_reads.pl** (Splits reads from the BWA-generated BAM between Read 1 and Read 2 and creates the two mpileup files using samtools mpileup) — https://github.com/Ettwiller/RIMS-seq/
- **samtools** (Called by split_mapped_reads.pl to generate the Read1/Read2 mpileup files with -Q 10 and -q 0 default filters)
- **mapping.pl** (Upstream step that uses BWA-mem to map paired-end reads to the reference genome and produce the BAM input) — https://github.com/Ettwiller/RIMS-seq/

## Examples

```
perl split_mapped_reads.pl -bam aln.bam -genome genome.fasta -mpileup1 read1.mpileup -mpileup2 read2.mpileup
```

## Evaluation signals

- Two distinct mpileup files are produced, one containing only Read 1 data and one only Read 2 data from the input BAM
- Default filters were applied: -Q 10 minimum base quality and -q 0 minimum mapping quality
- The genome FASTA passed to mpileup matches the genome used for BWA-mem mapping
- Downstream get_motif_all.pl runs successfully on both mpileup files and recovers expected motifs (e.g. CCWGG for E. coli K12; GATC and CGCG for A. calcoaceticus)
- C-to-T signals in Read 1 mpileups reflect the expected heat-alkaline deamination imbalance used for motif detection

## Limitations

- RIMS-seq analysis, including the Read1/Read2 split, does not work if the library preparation is based on tagmentation or other non-adapter-ligation technologies
- Adapters must be removed before mapping, otherwise the BAM handed to the splitting step is compromised
- For Option 2, the reference genome must be exactly the same as the sequenced bacterium; otherwise the SPAdes assembly route (Option 1) must be used

## Evidence

- [methods] split script: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] quality filters: "`-Q` : minimum base quality for a base to be considered (default=10)"
- [readme] library constraint: "RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies"
