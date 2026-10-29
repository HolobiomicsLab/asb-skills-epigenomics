---
name: base-quality-filtering-for-variant-counting
description: Use when counting C-to-T read variants (e.g. heat-alkaline deamination
  signals in RIMS-seq) from Read1/Read2 mpileups and background noise from sequencing
  errors must be suppressed via quality thresholds.
when_to_use_negative:
- Methylation motif discovery itself (get_motif_all.pl/MosDi step), where the mpileups
  are consumed for motif significance rather than variant counting
- Datasets where the library was prepared by tagmentation or other non-adapter-ligation
  protocols, since RIMS-seq deamination signal is not valid there
- Variant calling in polymorphic microbial communities when the goal is true SNV discovery,
  not deamination background suppression
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: split_mapped_reads.pl
  role: splits the bam into Read1 and Read2 mpileup files by invoking samtools mpileup
    where the -Q and -q quality thresholds are applied
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: samtools
  role: provides the mpileup command whose -Q (min base quality, default 10) and -q
    (min mapping quality, default 0) flags implement the filtering
- name: BWA
  role: BWA-mem mapping step (via mapping.pl) that precedes filtering; adapters must
    be removed beforehand
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:567f119b2514219f096a2130b1c9932cd9af15db9b1f83a25196ba5e30f0223c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - samtools
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# base-quality-filtering-for-variant-counting

## Summary

Applies samtools mpileup base-quality and mapping-quality thresholds plus a minimum base-calling quality score filter to limit background noise when counting C-to-T read variants from RIMS-seq Read1/Read2 mpileups.

## When to use

Use when counting C-to-T read variants (e.g. heat-alkaline deamination signals in RIMS-seq) from Read1/Read2 mpileups and background noise from sequencing errors must be suppressed via quality thresholds.

## When NOT to use

- Methylation motif discovery itself (get_motif_all.pl/MosDi step), where the mpileups are consumed for motif significance rather than variant counting
- Datasets where the library was prepared by tagmentation or other non-adapter-ligation protocols, since RIMS-seq deamination signal is not valid there
- Variant calling in polymorphic microbial communities when the goal is true SNV discovery, not deamination background suppression

## Inputs

- BAM file from BWA-mem paired-end mapping of RIMS-seq reads
- Read1 and Read2 samtools mpileup files (split_mapped_reads.pl output)
- Reference cytosine positions in the bacterial reference genome (e.g. Xp12)

## Outputs

- Quality-filtered Read1/Read2 mpileup variant counts at reference cytosines
- Per-condition R1/R2 C-to-T fold-imbalance and deamination-rate values (e.g. xp12_deamination_rates.csv)

## How to apply

After mapping RIMS-seq paired-end reads with BWA-mem (mapping.pl) and splitting the bam into Read1 and Read2 mpileups (split_mapped_reads.pl, which calls samtools mpileup), apply the mpileup filters -Q (minimum base quality, default=10) and -q (minimum mapping quality, default=0). Then, when counting C-to-T variants at reference cytosines, apply the base-calling quality score filter (--qualityscore, default=30; Illumina base-calling quality scores are ~30-35) so that only high-confidence mismatches are tallied. The rationale is that unfiltered mismatch counts are dominated by sequencing-error background, which would mask the genuine deamination imbalance between Read1 and Read2 (e.g. the 3.7-fold at 10 min up to 212-fold at 14 h in Xp12). Tune the qualityscore within the 30-35 Illumina range rather than below it, and count variants separately per read mate so the R1/R2 fold imbalance remains interpretable.

## Related tools

- **split_mapped_reads.pl** (splits the bam into Read1 and Read2 mpileup files by invoking samtools mpileup where the -Q and -q quality thresholds are applied) — https://github.com/Ettwiller/RIMS-seq/
- **samtools** (provides the mpileup command whose -Q (min base quality, default 10) and -q (min mapping quality, default 0) flags implement the filtering)
- **BWA** (BWA-mem mapping step (via mapping.pl) that precedes filtering; adapters must be removed beforehand) — https://github.com/lh3/bwa

## Examples

```
samtools mpileup -Q 10 -q 0 aln.bam | awk '...'   # equivalent to split_mapped_reads.pl, which calls samtools mpileup to produce Read1/Read2 mpileups
```

## Evaluation signals

- Recomputed R1/R2 C-to-T fold imbalance at 14 h matches the reported ~212-fold for Xp12
- Recomputed deamination rates fall in the reported 0.24% (0.1 M NaOH, 3 h) to 2.72% (0.5 M NaOH, 3 h) range as plotted in Supplementary Figure S2C
- Fold imbalance at short treatment (10 min) is recoverable (~3.7-fold), indicating the quality filters kept error background below the deamination signal
- Only cytosine positions with bases passing the minimum base-calling quality score (default 30, Illumina range 30-35) contribute to variant counts

## Limitations

- Single nucleotide polymorphisms in microbial communities may confound C-to-T deamination detection, increasing background noise even with quality filters
- RIMS-seq signals require enough read coverage; low-coverage or rare-motif sites may not yield detectable signal after quality filtering
- The default qualityscore of 30 is near the Illumina base-calling ceiling (~30-35); setting it too high would discard nearly all bases

## Evidence

- [methods] minimum base quality: "`-Q` : minimum base quality for a base to be considered (default=10)"
- [methods] minimum mapping quality: [evidence span withheld — anchor status: not_found]
- [methods] base calling quality score: "`--qualityscore` : minimum base calling quality score (default=30)."
- [methods] mpileup splitting: "Reads are separated between Read 1 and Read 2 and two files are created accordingly. This is done using samtools mpileup."
- [discussion] SNP confounding: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise"
