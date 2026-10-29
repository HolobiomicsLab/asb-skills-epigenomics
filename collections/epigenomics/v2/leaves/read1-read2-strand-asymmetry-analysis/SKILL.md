---
name: read1-read2-strand-asymmetry-analysis
description: Use when paired-end bacterial sequencing reads from heat-alkaline-treated
  (RIMS-seq) vs regular DNA-seq libraries must be checked for mate-specific (R1 vs
  R2) C-to-T elevation in a methylated motif context such as CCWGG.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or another non-adapter-ligation
  protocol — RIMS-seq does not work for such libraries
- The reference genome is not exactly the same as the sequenced bacterium and no in-pipeline
  assembly (SPAdes option) is available
- The goal is de novo methylase specificity/motif discovery (get_motif_all.pl with
  MoSDi) rather than context-resolved C-to-T fraction quantification
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3174
tools:
- name: mapping.pl
  role: Maps paired-end reads to the reference genome using BWA-mem; adapters must
    be removed prior to use
  repo: github.com/Ettwiller/RIMS-seq
- name: split_mapped_reads.pl
  role: Splits the bam from BWA into Read 1 and Read 2 mpileup files using samtools
    mpileup, honoring -Q minimum base quality
  repo: github.com/Ettwiller/RIMS-seq
- name: samtools
  role: mpileup engine used by split_mapped_reads.pl to separate reads and apply base-quality
    filtering
  repo: http://www.htslib.org/download/
- name: BWA
  role: Provides the BWA-mem aligner invoked by mapping.pl for paired-end mapping
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:a907e34212f6976bc9d93bf9a478e1cee1f33aa5ec4642c9232548c2314285d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - mapping.pl
  - samtools
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Read 1 vs Read 2 strand asymmetry analysis of C-to-T variants in sequence contexts

## Summary

Splits mapped paired-end RIMS-seq reads into Read 1 and Read 2 mpileups and compares C-to-T read-variant fractions between the dcm CCWGG context (W = A or T) and all other C contexts, per read mate and per strain. The R1-only, dcm-dependent elevation is the diagnostic signal of heat-alkaline-induced deamination at methylated cytosines.

## When to use

Use when paired-end bacterial sequencing reads from heat-alkaline-treated (RIMS-seq) vs regular DNA-seq libraries must be checked for mate-specific (R1 vs R2) C-to-T elevation in a methylated motif context such as CCWGG.

## When NOT to use

- The sequencing library was prepared by tagmentation or another non-adapter-ligation protocol — RIMS-seq does not work for such libraries
- The reference genome is not exactly the same as the sequenced bacterium and no in-pipeline assembly (SPAdes option) is available
- The goal is de novo methylase specificity/motif discovery (get_motif_all.pl with MoSDi) rather than context-resolved C-to-T fraction quantification

## Inputs

- Paired-end fastq.gz reads (e.g. ecoli_rims_3h_5M.1.fastq.gz / .2.fastq.gz, 5 million read pairs)
- Reference genome FASTA identical to the sequenced bacterium (e.g. E. coli K12 MG1655 from the RIMS-seq assembly)
- Bam file produced by BWA-mem mapping (adapters removed beforehand)

## Outputs

- Read 1 and Read 2 mpileup files generated with samtools mpileup at -Q 35
- Per-context C-to-T variant counts (first C of CCWGG vs all other C contexts) per read mate
- ccwgg_ct_fractions.csv table of CCWGG vs other-context C-to-T fractions by read (R1, R2) and strain (dcm+ K12, dcm- BL21)

## How to apply

Downsample paired-end fastq.gz reads to 1-5 million pairs to avoid lengthy processing, then map with BWA-mem via mapping.pl against a reference genome that is exactly the same as the sequenced bacterium (or an assembly derived in the pipeline). Split the resulting bam into Read 1 and Read 2 mpileup files with split_mapped_reads.pl, which uses samtools mpileup, applying a minimum base quality of -Q 35 so only high-confidence C-to-T calls are retained. For each mpileup, classify every C position by sequence context — first C of CCWGG versus all other C contexts — and count C-to-T read variants per context. Compute the CCWGG vs other-context C-to-T fractions separately for R1 and R2 and for dcm+ (E. coli K12) and dcm- (BL21) strains, writing results to ccwgg_ct_fractions.csv. The key decision point is the R1-only test: a genuine RIMS-seq signal appears in R1 of the dcm+ strain but not in R2, not in the dcm- strain, and not in regular DNA-seq.

## Related tools

- **mapping.pl** (Maps paired-end reads to the reference genome using BWA-mem; adapters must be removed prior to use) — github.com/Ettwiller/RIMS-seq
- **split_mapped_reads.pl** (Splits the bam from BWA into Read 1 and Read 2 mpileup files using samtools mpileup, honoring -Q minimum base quality) — github.com/Ettwiller/RIMS-seq
- **samtools** (mpileup engine used by split_mapped_reads.pl to separate reads and apply base-quality filtering) — http://www.htslib.org/download/
- **BWA** (Provides the BWA-mem aligner invoked by mapping.pl for paired-end mapping) — https://github.com/lh3/bwa

## Examples

```
samtools mpileup -Q 35 aln-pe.bam
```

## Evaluation signals

- K12 (dcm+) 3h R1 CCWGG C-to-T fraction reaches ~25%, well above the ~1.9% regular DNA-seq baseline
- No CCWGG-specific elevation appears in R2 of K12, in either mate of dcm- BL21, or in regular DNA-seq — the signal is R1- and dcm-dependent
- Both R1 and R2 mpileup files exist and only bases passing -Q 35 (high-confidence calls) contribute counts
- CCWGG is classified strictly as the first C of CCWGG with W = A or T; all other C contexts are counted separately as background
- Total C-to-T counts in CCWGG and other contexts sum to the per-mate totals in ccwgg_ct_fractions.csv (internal consistency check)

## Limitations

- SNPs in microbial communities can confound C-to-T deamination detection and increase background noise, so single-strain references or assemblies are preferred
- Partially methylated sites yield weak signals; RIMS-seq needs enough read coverage to compensate for the scarcity of motif sites
- Only a -Q 35 (high-confidence) call set is interpretable here; the pipeline default of -Q 10 is too permissive for the CCWGG fraction comparison

## Evidence

- [methods] This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly.: "This script splits the reads from the bam file generated by BWA. Reads are separated between Read 1 and Read 2 and two files are created accordingly."
- [methods] This is done using samtools mpileup.: "This is done using samtools mpileup."
- [methods] `-Q` : minimum base quality for a base to be considered (default=10): "`-Q` : minimum base quality for a base to be considered (default=10)"
- [discussion] single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise: "single nucleotide polymorphisms found in microbial communities may confound the identification of the C to T deamination, increasing the background noise"
- [readme] RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies: "RIMS-seq does not work if the library preparation protocol is based on tagmentation or other technologies"
