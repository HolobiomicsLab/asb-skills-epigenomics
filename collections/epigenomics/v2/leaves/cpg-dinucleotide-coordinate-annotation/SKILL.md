---
name: cpg-dinucleotide-coordinate-annotation
description: Use when direct-BSP or cloning-BSP trace files (.ab1) must have their
  cytosine/CpG positions converted to genomic coordinates from a reference .fasta
  template, as in ABSP individual sample methylation analysis.
when_to_use_negative:
- Input is high-throughput bisulfite sequencing (FASTQ/aligner-based) data rather
  than .ab1 trace files from BSP PCR products
- Only CpG-numbered or relative positions are needed and no genomic coordinate annotation
  is required (ABSP can label by 'CpG numbers' or 'None' in Grouped Analysis)
- The reference sequence has no real genomic location and arbitrary coordinates were
  assigned — then the computed coordinates must not be interpreted as genomic
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: ABSP
  role: R tool that performs the local pair-wise alignment and applies the offset
    formula to annotate CpG coordinates on trimmed reads
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Language in which the ABSP individual analysis (ABSP_individual_analysis.Rmd,
    ABSP_functions.R) is implemented
- name: Gviz
  role: Used for genomic visualization plots of the reference sequence track alongside
    the annotated coordinates
- name: Shiny
  role: User interface through which the Individual analysis tab is run with the .fasta
    and .ab1 inputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:ed827abfa2a073533b4021e8507158f14bbd9a8c69c1ac1a998e5fb4ab74bb79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - Gviz
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cpg-dinucleotide-coordinate-annotation

## Summary

Annotates cytosines in bisulfite sequencing PCR reads with genomic CpG coordinates by local pair-wise alignment of each trimmed .ab1 read against the reference DNA template, using the alignment offset formula (read position = template position − Sstart + Pstart). This lets ABSP report methylation percentages at genomic coordinates rather than read-local positions.

## When to use

Use when direct-BSP or cloning-BSP trace files (.ab1) must have their cytosine/CpG positions converted to genomic coordinates from a reference .fasta template, as in ABSP individual sample methylation analysis.

## When NOT to use

- Input is high-throughput bisulfite sequencing (FASTQ/aligner-based) data rather than .ab1 trace files from BSP PCR products
- Only CpG-numbered or relative positions are needed and no genomic coordinate annotation is required (ABSP can label by 'CpG numbers' or 'None' in Grouped Analysis)
- The reference sequence has no real genomic location and arbitrary coordinates were assigned — then the computed coordinates must not be interpreted as genomic

## Inputs

- Reference DNA sequence in .fasta format with genomic coordinates in the header (e.g. 'chr1:1-XXX')
- Raw sequencing trace files in .ab1 format (chromatograms), trimmed by Phred-quality and primary-peak-ratio methods
- Optional reference genome assembly (e.g. GCF_900626175.2_cs10) for genomic plots only

## Outputs

- Coordinate-annotated CpG methylation table (methylated vs unmethylated counts per CpG site) written to the results folder
- Per-sample .html report in the reports folder including the methylation table

## How to apply

Load the reference sequence from the .fasta file (coordinates such as 'chr1:1-200' are parsed from the FASTA header) and the raw chromatograms from the .ab1 files. Scan the reference template to identify all CpG dinucleotide positions. Align each trimmed read to the reference template with a local pair-wise alignment, choosing the sense or antisense template based on the longest alignment, and derive the alignment offset from the reference start/end coordinates. Convert each read cytosine to a genomic coordinate as: cytosine position on template − SstartF + PstartF (e.g. 100 − 30 + 22 = 92 on the forward strand). Tally methylated versus unmethylated cytosines at each CpG and write the coordinate-annotated methylation table to the results folder; if no CpG sites are covered by the sequencing results after trimming, the run must stop with the error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'

## Related tools

- **ABSP** (R tool that performs the local pair-wise alignment and applies the offset formula to annotate CpG coordinates on trimmed reads) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Language in which the ABSP individual analysis (ABSP_individual_analysis.Rmd, ABSP_functions.R) is implemented)
- **Gviz** (Used for genomic visualization plots of the reference sequence track alongside the annotated coordinates)
- **Shiny** (User interface through which the Individual analysis tab is run with the .fasta and .ab1 inputs)

## Examples

```
RStudio: open app.R -> click "Run App" -> "Individual analysis" tab -> add reference .fasta and .ab1 files -> run (equivalently: shiny::runApp("app.R"))
```

## Evaluation signals

- Spot-check the offset arithmetic: a cytosine at template position 100 with SstartF=30 and PstartF=22 must map to read/genomic position 92 on the forward strand
- The output methylation table must contain at least one covered CpG site; zero covered CpGs must trigger the abort error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- Annotated coordinates must fall within the coordinate range declared in the reference FASTA header (e.g. chr1:1-200 for a 200 bp sequence)
- Sense vs antisense template choice must correspond to the longest local pair-wise alignment of the trimmed read
- Final trimmed sequence length must be consistent with the combined quality-score (orange) and primary-ratio (blue) trimming shown in the Trimming plot

## Limitations

- The reference genome is only necessary for visualization in genomic plots and is not needed to compute methylation percentages; with an arbitrary genome, coordinates in tables/plots must not be taken into consideration
- Custom reporter sequences not present in a reference genome require arbitrary FASTA-header coordinates (e.g. 'chr1:1-200') matching the sequence length, and 'CpG numbers' or 'None' label options are recommended
- Over-aggressive trimming (default th_quality_error=0.001, i.e. Phred 30; th_mixed_position=0.75; th_mixed_perc=75%) can leave the sequence too short with few covered CpG sites or abort the coordinate annotation step

## Evidence

- [other] ABSP maps cytosine positions from the template DNA onto the sequencing read using the local pair-wise alignment offset: cytosine position = cytosine position on template − SstartF + PstartF (e.g. 100 − 30 + 22 = 92 on the forward strand).: [evidence span withheld — anchor status: not_found]
- [abstract] Abstract statement on raw inputs and outputs of the methylation computation.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [readme] README description of the abort condition when no CpG sites are covered.: "the analysis may be aborted with the error message: “Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.”"
- [readme] README clarification that the reference genome is only for visualization, not for methylation coordinate computation.: "the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages"
