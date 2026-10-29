---
name: c-to-t-deamination-rate-quantification
description: Use when paired-end reads from heat-alkaline (NaOH)-treated DNA are available
  and you need per-condition C-to-T deamination rates or the Read1/Read2 fold imbalance
  at reference cytosines.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or another non-adapter-ligation
  protocol, which the article states breaks the RIMS-seq approach
- Methylation analysis uses EM-seq or bisulfite-seq, which already give the precise
  genomic location of modified cytosines without relying on R1/R2 deamination asymmetry
- You only need genome assembly or shotgun QC metrics and have no interest in deamination
  quantification or methylase specificity
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mapping.pl
  role: Maps adapter-trimmed paired-end reads to the reference genome in paired-end
    mode using BWA-mem, producing the BAM that split_mapped_reads.pl consumes
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: split_mapped_reads.pl
  role: Splits the BAM into Read1 and Read2 mpileup files via samtools mpileup so
    C-to-T variants can be counted per mate (-Q minimum base quality 10, -q minimum
    mapping quality 0)
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: BWA
  role: Alignment engine invoked by mapping.pl; `bwa mem` performs the paired-end
    mapping to the reference genome
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:e264d8de69940d38dde6bd164d245e91f259f48c191aa5f733e10ad56dcde89d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - mapping.pl
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# c-to-t-deamination-rate-quantification

## Summary

Quantifies heat-alkaline deamination of m5C sites by counting C-to-T read variants separately in Read1 and Read2 mpileups, yielding the R1/R2 fold-imbalance (up to 212-fold at 14 h in Xp12 DNA) and the deamination rate per NaOH condition (0.24%–2.72%). It converts RIMS-seq's deamination chemistry into a quantitative time-course and rate table.

## When to use

Use when paired-end reads from heat-alkaline (NaOH)-treated DNA are available and you need per-condition C-to-T deamination rates or the Read1/Read2 fold imbalance at reference cytosines.

## When NOT to use

- The sequencing library was prepared by tagmentation or another non-adapter-ligation protocol, which the article states breaks the RIMS-seq approach
- Methylation analysis uses EM-seq or bisulfite-seq, which already give the precise genomic location of modified cytosines without relying on R1/R2 deamination asymmetry
- You only need genome assembly or shotgun QC metrics and have no interest in deamination quantification or methylase specificity

## Inputs

- Paired-end Illumina FASTQ files (adapter-ligation libraries) from NaOH-treated DNA, e.g. BioProject PRJNA706563 Xp12 read sets
- Reference genome FASTA identical to the sequenced organism (or a SPAdes-assembled genome)
- BAM alignment produced by mapping.pl (BWA-mem, adapters removed beforehand)

## Outputs

- Read1 and Read2 samtools mpileup files per condition (minimum base quality Q=10, minimum mapping quality q=0)
- Per-condition R1/R2 C-to-T fold-imbalance values (e.g. 3.7-fold at 10 min to 212-fold at 14 h)
- Per-condition deamination rates (fraction of cytosines converted to thymine, e.g. 0.24%–2.72%)
- CSV table of fold-imbalance and deamination-rate values (xp12_deamination_rates.csv)

## How to apply

Download the paired-end FASTQ sets for each incubation time (0–14 h) and NaOH concentration (e.g. PRJNA706563 Xp12 data), remove adapters, and map read pairs to the matching reference genome with BWA-mem via mapping.pl (the exact reference genome is required; otherwise assemble with SPAdes first). Split each BAM into Read1 and Read2 mpileups using split_mapped_reads.pl, which calls samtools mpileup with minimum base quality Q=10 and minimum mapping quality q=0. For each condition, count C-to-T read variants at reference cytosines separately in the R1 and R2 mpileups, applying the base-calling quality filter (--qualityscore, default 30; 30–35 for Illumina) to suppress background noise. The rationale for splitting by mate is that deamination during the 3 h NaOH incubation is fixed in Read1, whereas Read2 is copied afterwards, so the R1/R2 C-to-T ratio reports methylated-cytosine deamination. Compute the R1/R2 fold imbalance per incubation time (3.7-fold at 10 min rising to 212-fold at 14 h) and the deamination rate per NaOH condition (0.24% at 0.1 M NaOH/3 h up to 2.72% at 0.5 M NaOH/3 h), then write per-condition values to a CSV such as xp12_deamination_rates.csv.

## Related tools

- **mapping.pl** (Maps adapter-trimmed paired-end reads to the reference genome in paired-end mode using BWA-mem, producing the BAM that split_mapped_reads.pl consumes) — https://github.com/Ettwiller/RIMS-seq/
- **split_mapped_reads.pl** (Splits the BAM into Read1 and Read2 mpileup files via samtools mpileup so C-to-T variants can be counted per mate (-Q minimum base quality 10, -q minimum mapping quality 0)) — https://github.com/Ettwiller/RIMS-seq/
- **BWA** (Alignment engine invoked by mapping.pl; `bwa mem` performs the paired-end mapping to the reference genome) — https://github.com/lh3/bwa

## Examples

```
./bwa mem ref.fa read1.fq read2.fq | gzip -3 > aln-pe.sam.gz
```

## Evaluation signals

- Recomputed R1/R2 C-to-T fold imbalance at 14 h matches the reported ~212-fold value
- Recomputed deamination rates fall within the reported 0.24%–2.72% range, as plotted in Supplementary Figure S2C
- The fold imbalance increases monotonically with heat-alkaline treatment time across the 0–14 h series
- Read1 and Read2 C-to-T counts are near parity in the untreated (0 h) control, so mate asymmetry only emerges after NaOH incubation
- xp12_deamination_rates.csv contains one fold-imbalance and one rate value per time/NaOH condition with no missing entries

## Limitations

- Single nucleotide polymorphisms, especially in microbial communities, can confound the C-to-T deamination signal and increase background noise, so this quantification is most reliable on clonal DNA or exact reference-matched samples
- RIMS-seq signals require enough read coverage over sites; low-coverage conditions will not yield reliable fold-imbalance estimates
- The protocol requires adapter-ligation libraries and the column-based cleaning step; skipping the cleaning step (amplifying a small library volume) has not been tested

## Evidence

- [methods] This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly.: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] `--qualityscore` : minimum base calling quality score (default=30).: "`--qualityscore` : minimum base calling quality score (default=30)."
- [methods] -Q : minimum base quality for a base to be considered (default=10): "-Q : minimum base quality for a base to be considered (default=10)"
- [supplementary] The data have been deposited with links to BioProject accession number PRJNA706563: "The data have been deposited with links to BioProject accession number PRJNA706563 in the NCBI BioProject database"
- [discussion] single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise"
- [methods] This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script.: "This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script."
