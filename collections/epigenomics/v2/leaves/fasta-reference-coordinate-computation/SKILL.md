---
name: fasta-reference-coordinate-computation
description: Use when direct-BSP or cloning-BSP reads (.ab1 trace files) must be mapped
  onto a .fasta reference template so each read cytosine receives a genomic coordinate
  and per-CpG methylation levels can be tallied.
when_to_use_negative:
- The goal is only visualization in genomic plots — the README states 'the reference
  genome is only necessary for visualization of the reference sequence in genomic
  plots but it is not needed for computing methylation percentages', so do not block
  coordinate computation on genome availability
- Input data are not BSP trace files (e.g. already bisulfite-converted FASTQ reads)
  — ABSP is designed for raw .ab1 chromatograms from direct-BSP and cloning-BSP
- No CpG sites are covered by the trimmed sequencing results — the analysis must be
  stopped rather than produce an empty coordinate table
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ABSP
  role: R tool that performs the local pair-wise alignment of trimmed reads against
    the .fasta template, computes the Sstart/Pstart offset, and emits the coordinate-annotated
    methylation table
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP's individual sample analysis and coordinate
    computation are executed
  repo: https://www.r-project.org/
- name: RStudio
  role: IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the Shiny
    app that drives the Individual analysis
  repo: https://www.rstudio.com/products/rstudio/download/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:9686aad36879aa4226876435db65fe7030c5d25b60c64bfa11a649154ea4ee70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# fasta-reference-coordinate-computation

## Summary

Computation of CpG cytosine coordinates from a .fasta reference DNA and their projection onto trimmed .ab1 bisulfite sequencing reads via the local pair-wise alignment offset in ABSP, so methylation can be reported per genomic CpG position.

## When to use

Use when direct-BSP or cloning-BSP reads (.ab1 trace files) must be mapped onto a .fasta reference template so each read cytosine receives a genomic coordinate and per-CpG methylation levels can be tallied.

## When NOT to use

- The goal is only visualization in genomic plots — the README states 'the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages', so do not block coordinate computation on genome availability
- Input data are not BSP trace files (e.g. already bisulfite-converted FASTQ reads) — ABSP is designed for raw .ab1 chromatograms from direct-BSP and cloning-BSP
- No CpG sites are covered by the trimmed sequencing results — the analysis must be stopped rather than produce an empty coordinate table

## Inputs

- Reference sequence in .fasta format (e.g. CDH1)
- Raw sequencing chromatogram files in .ab1 format
- Trimmed read sequences produced by ABSP's sequencing-trimming step (Phred-quality and primary-peak-ratio based)

## Outputs

- Coordinate-annotated CpG methylation table written to the 'results' folder
- Methylated versus unmethylated cytosine counts per covered CpG site

## How to apply

Load the reference sequence from the CDH1-style .fasta file and the raw chromatograms from the .ab1 files in the ABSP 'Individual analysis' tab. Scan the reference to enumerate all CpG dinucleotide positions, then align each trimmed read to the reference template with a local pair-wise alignment, selecting the sense or antisense template by the longest alignment. Convert read cytosine positions to template/genomic coordinates using the alignment offset derived from the reference start/end coordinates: cytosine position = cytosine position on template − SstartF + PstartF (e.g. 100 − 30 + 22 = 92 on the forward strand). Tally methylated versus unmethylated cytosines at each covered CpG and write the coordinate-annotated methylation table to the 'results' folder; if no CpG sites are covered, the run must be stopped with the error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'

## Related tools

- **ABSP** (R tool that performs the local pair-wise alignment of trimmed reads against the .fasta template, computes the Sstart/Pstart offset, and emits the coordinate-annotated methylation table) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP's individual sample analysis and coordinate computation are executed) — https://www.r-project.org/
- **RStudio** (IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the Shiny app that drives the Individual analysis) — https://www.rstudio.com/products/rstudio/download/

## Examples

```
cs10 <- readDNAStringSet("/PATH/TO/FOLDER/GCF_900626175.2_cs10_genomic.fna.gz")
```

## Evaluation signals

- A methylation table with covered CpG sites is present in the 'results' folder after the run
- Spot-check the offset arithmetic: a cytosine at template position 100 with SstartF=30 and PstartF=22 must be reported at position 92 on the forward strand
- Each read cytosine coordinate must equal its template position minus Sstart plus Pstart derived from the reference .fasta header coordinates
- If no CpG sites are covered, the run must abort with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- The .html report in the 'reports' folder lists the same covered CpG coordinates as the methylation table

## Limitations

- Arbitrary or mismatched reference genomes affect only visualization and the coordinates shown in plots/tables — with an arbitrary genome, coordinates must not be taken into consideration and 'CpG numbers' or 'None' CpG label options are recommended in Grouped Analysis
- Over-aggressive trimming (Phred quality method with th_quality_error=0.001, or primary-peak-ratio method with th_mixed_position=0.75 / th_mixed_perc=75%) can leave too few CpG sites covered and abort the analysis
- Custom reporter sequences not present in a reference genome require arbitrary coordinates (e.g. 'chr1:1-200') in the .fasta header matching the sequence length

## Evidence

- [other] ABSP maps cytosine positions from the template DNA onto the sequencing read using the local pair-wise alignment offset: cytosine position = cytosine position on template − SstartF + PstartF (e.g. 100 − 30 + 22 = 92 on the forward strand).: [evidence span withheld — anchor status: not_found]
- [readme] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format.: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
- [readme] First, the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages.: "the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages"
- [readme] You may either end up with a very short final sequence with only a few CpG sites covered in the “Methylation” tab, or the analysis may be aborted with the error message: “Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.”: "Error: Analysis has been stopped as no CpG sites were found covered by sequencing results."
