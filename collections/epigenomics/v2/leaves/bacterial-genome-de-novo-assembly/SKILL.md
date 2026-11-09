---
name: bacterial-genome-de-novo-assembly
description: Use when paired-end Illumina-style bacterial reads (e.g. RIMS-seq FASTQ
  pairs from PRJNA706563) must be turned into a reference-free genome assembly, or
  when alkaline-treated reads must be shown to assemble as well as a DNA-seq(+3H)
  control.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or another non-adapter-ligation
  protocol — RIMS-seq does not work in that case
- An exactly matching reference genome for the sequenced strain is available and a
  reference-based mapping (RIMS-seq option 2) is preferred over assembly
- Reads are long-read (PacBio/Nanopore) rather than short paired-end reads
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: SPAdes
  role: performs the de novo genome assembly of trimmed paired-end reads, producing
    the contigs FASTA used as the reference in RIMS-seq option 1
  repo: https://github.com/ablab/spades
- name: Quast
  role: evaluates the RIMS-seq vs control assemblies, extracting mismatch rate and
    N50 for the quality comparison
- name: BWA-mem
  role: maps the paired-end reads back to the assembled genome in paired-end mode
    to generate the .bam input for the downstream motif-discovery pipeline
  repo: https://github.com/lh3/bwa
- name: RIMS-seq.pl
  role: wrapper that automates the option-1 workflow (SPAdes assembly, mapping.pl,
    split_mapped_reads.pl, get_motif_all.pl) from paired-end reads
  repo: https://github.com/Ettwiller/RIMS-seq/
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:a7c92896abcd4ca1abee3fe77e36c8d55b56e6aacfbf2a3a83f8d4118d57beea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - BWA-mem
  - Quast
  - RIMS-seq.pl
  - SPAdes
  - split_mapped_reads.pl
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# bacterial-genome-de-novo-assembly

## Summary

De novo assembles a bacterial genome directly from paired-end sequencing reads (e.g. RIMS-seq alkaline-treated reads) with SPAdes, so that no reference genome is required, and evaluates assembly quality (N50, mismatch rate) against a standard shotgun control.

## When to use

Use when paired-end Illumina-style bacterial reads (e.g. RIMS-seq FASTQ pairs from PRJNA706563) must be turned into a reference-free genome assembly, or when alkaline-treated reads must be shown to assemble as well as a DNA-seq(+3H) control.

## When NOT to use

- The sequencing library was prepared by tagmentation or another non-adapter-ligation protocol — RIMS-seq does not work in that case
- An exactly matching reference genome for the sequenced strain is available and a reference-based mapping (RIMS-seq option 2) is preferred over assembly
- Reads are long-read (PacBio/Nanopore) rather than short paired-end reads

## Inputs

- paired-end FASTQ reads (R1/R2, e.g. gzipped) from a RIMS-seq or standard DNA-seq library
- adapter-trimmed read pairs (trimming required before downstream mapping)
- optional: DNA-seq(+3H) control paired-end reads for quality comparison

## Outputs

- contigs FASTA (SPAdes de novo assembly usable as reference for RIMS-seq option 1)
- Quast assembly metrics (N50, mismatch rate), e.g. side-by-side in quast_assembly_comparison.tsv

## How to apply

Obtain paired-end FASTQ files (e.g. from NCBI BioProject PRJNA706563), perform adapter and quality trimming, then run SPAdes to produce a contigs FASTA; for RIMS-seq workflows this assembly becomes the reference used downstream by RIMS-seq.pl (option 1). In the A. calcoaceticus ATCC 49823 comparison, assemble both the RIMS-seq (alkaline-treated) library and the DNA-seq(+3H) control, then evaluate each with Quast to extract mismatch rate and N50. Validation rests on the RIMS-seq assembly matching the control on these metrics, per Figure 3B and Supplementary Table S1. Note the README caveat that the library must be adapter-ligation based, and that when a reference is used instead of assembly (option 2) it must be exactly the same bacterial strain.

## Related tools

- **SPAdes** (performs the de novo genome assembly of trimmed paired-end reads, producing the contigs FASTA used as the reference in RIMS-seq option 1) — https://github.com/ablab/spades
- **Quast** (evaluates the RIMS-seq vs control assemblies, extracting mismatch rate and N50 for the quality comparison)
- **BWA-mem** (maps the paired-end reads back to the assembled genome in paired-end mode to generate the .bam input for the downstream motif-discovery pipeline) — https://github.com/lh3/bwa
- **RIMS-seq.pl** (wrapper that automates the option-1 workflow (SPAdes assembly, mapping.pl, split_mapped_reads.pl, get_motif_all.pl) from paired-end reads) — https://github.com/Ettwiller/RIMS-seq/

## Examples

```
spades.py --pe1-1 RIMS_R1.fastq.gz --pe1-2 RIMS_R2.fastq.gz -o rims_assembly
```

## Evaluation signals

- The RIMS-seq assembly's mismatch rate and N50 are comparable to those of the DNA-seq(+3H) control assembly (Quast, Figure 3B / Supplementary Table S1)
- quast_assembly_comparison.tsv contains side-by-side RIMS-seq vs control Quast metrics
- The assembled contigs FASTA supports downstream recovery of methylated motifs when reads are mapped to it (GATC P = 1.44e-1255, CGCG P = 8.6e-228 for A. calcoaceticus), matching the reference-genome results
- Assembly is derived only from adapter/quality-trimmed paired-end reads, with no reference genome supplied (RIMS-seq option 1 behavior)
- Sequencing QC metrics (coverage, GC content, mapping rate) are similar between the RIMS-seq and DNA-seq assemblies

## Limitations

- The de novo assembly can only be benchmarked for motif recovery against a reference-genome run; when the exact reference genome is unavailable, option 1 (assembly-derived reference) is recommended
- Assembly quality claims are validated for specific strains (e.g. A. calcoaceticus ATCC 49823); other genomes may not show unchanged mismatch rate/N50
- RIMS-seq library preparation must be based on adapter ligation; tagmentation-based protocols are incompatible

## Evidence

- [results] De novo assembly of RIMS-seq reads of A. calcoaceticus ATCC 49823 shows that the alkaline treatment did not alter key assembly-quality metrics such as mismatch rate and N50 compared to the DNA-seq(+3H) control.: [evidence span withheld — anchor status: not_found]
- [methods] It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis.: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [intro] RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing.: "RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing."
- [readme] Note: if you have a reference genome and perform you own mapping, SPAdes and BWA are not necessary (see option 2).: "Note: if you have a reference genome and perform you own mapping, SPAdes and BWA are not necessary (see option 2)."
- [readme] the sequencing library should be done using a protocol based on adapter ligation: "Note that the sequencing library should be done using a protocol based on adapter ligation."
