---
name: sense-antisense-template-selection-by-longest-alignment
description: Use when analyzing direct-BSP or cloning-BSP chromatogram traces (.ab1)
  against a reference .fasta and you must decide read orientation and map cytosine
  positions onto CpG coordinates before computing methylation percentages.
when_to_use_negative:
- Input is not BSP chromatogram data (e.g. Illumina bisulfite FASTQ reads) — ABSP
  expects raw .ab1 trace files
- The reference genome or sequence is unavailable for visualization only; note that
  methylation computation itself does not require the reference genome, so this skill
  is still applicable, but displayed genomic coordinates from an arbitrary genome
  must not be interpreted
- You only need group-level comparison or lollipop plots without per-read CpG coordinate
  mapping (use the Grouped analysis tab instead)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3295
tools:
- name: ABSP
  role: R tool implementing the local pair-wise alignment to sense/antisense templates,
    longest-alignment strand selection, and the Sstart/Pstart offset-based cytosine-to-genomic-coordinate
    mapping
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment for ABSP; installed alongside RStudio to run the ABSP_individual_analysis.Rmd
    workflow
  repo: https://www.r-project.org/
- name: Shiny
  role: Built-in Shiny app interface exposing the 'Individual analysis' tab where
    the .fasta and .ab1 inputs are supplied
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:6c25d71de788ea2b1e3bd8e0636aa1f24a73ee68392d6f646c30f65800b1340d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# sense-antisense-template-selection-by-longest-alignment

## Summary

Within ABSP's individual sample analysis, each trimmed .ab1 sequencing read is locally pair-wise aligned to both the sense and antisense versions of the reference (e.g. CDH1 .fasta) template, and the orientation yielding the longest alignment is chosen so cytosine/CpG positions can be mapped onto the read via the alignment offset (template position − Sstart + Pstart).

## When to use

Use when analyzing direct-BSP or cloning-BSP chromatogram traces (.ab1) against a reference .fasta and you must decide read orientation and map cytosine positions onto CpG coordinates before computing methylation percentages.

## When NOT to use

- Input is not BSP chromatogram data (e.g. Illumina bisulfite FASTQ reads) — ABSP expects raw .ab1 trace files
- The reference genome or sequence is unavailable for visualization only; note that methylation computation itself does not require the reference genome, so this skill is still applicable, but displayed genomic coordinates from an arbitrary genome must not be interpreted
- You only need group-level comparison or lollipop plots without per-read CpG coordinate mapping (use the Grouped analysis tab instead)

## Inputs

- Reference sequence in .fasta format (e.g. CDH1), with genomic coordinates in the header (e.g. chr1:1-XXX)
- Raw Sanger sequencing trace files in .ab1 format (direct-BSP or cloning-BSP)
- Trimmed sequencing reads produced by ABSP's Phred-quality and primary-peak-ratio trimming steps

## Outputs

- Coordinate-annotated CpG methylation table per sample in the results folder
- CpG methylation levels (methylated vs unmethylated cytosine counts per CpG site)

## How to apply

Load the reference sequence from the CDH1 .fasta file and the raw chromatograms from the .ab1 files in the ABSP 'Individual analysis' tab. Scan the reference to identify all CpG dinucleotide positions, then perform a local pair-wise alignment of each trimmed read against both the sense and antisense template and select the template giving the longest alignment as the correct strand. Using the reference start/end coordinates embedded in the FASTA header, compute each read cytosine's genomic coordinate as: cytosine position on template − Sstart + Pstart (e.g. 100 − 30 + 22 = 92 on the forward strand). Tally methylated versus unmethylated cytosines at each covered CpG, write the coordinate-annotated methylation table to the results folder, and stop with the error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.' if no CpG site is covered.

## Related tools

- **ABSP** (R tool implementing the local pair-wise alignment to sense/antisense templates, longest-alignment strand selection, and the Sstart/Pstart offset-based cytosine-to-genomic-coordinate mapping) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment for ABSP; installed alongside RStudio to run the ABSP_individual_analysis.Rmd workflow) — https://www.r-project.org/
- **Shiny** (Built-in Shiny app interface exposing the 'Individual analysis' tab where the .fasta and .ab1 inputs are supplied) — https://github.com/ABSP-methylation-tool/ABSP

## Examples

```
Open ABSP Rproject.Rproject in RStudio and run app.R → 'Individual analysis' tab → supply CDH1 reference .fasta + sample .ab1 files → run analysis (ABSP_individual_analysis.Rmd)
```

## Evaluation signals

- The output methylation table contains covered CpG sites; an empty run must abort with 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- Spot-check the offset arithmetic: a cytosine at template position 100 with SstartF = 30 and PstartF = 22 must map to read position 92 on the forward strand (template position − Sstart + Pstart).
- Each read's methylation calls are anchored to the template chosen by the longest local pair-wise alignment; the .html report in the 'reports' folder should reflect strand-consistent CpG coordinates matching the FASTA header coordinates.
- The coordinate-annotated methylation table (e.g. CDH1_high methylated DNA_3_meth_table) appears in the 'results' folder after the analysis completes.

## Limitations

- If trimming is too aggressive, the trimmed sequence may be too short, leaving few CpG sites covered or aborting the analysis with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.' — adjust th_quality_error (default 0.001, line 607) or th_mixed_position (default 0.75, line 612).
- Using an arbitrary reference genome or custom reporter sequence with arbitrary coordinates means the displayed genomic coordinates in tables and plots must not be taken into consideration.
- The reference genome is only needed for genomic visualization; computing methylation percentages does not require it.

## Evidence

- [other] ABSP maps cytosine positions from the template DNA onto the sequencing read using the local pair-wise alignment offset: cytosine position = cytosine position on template − SstartF + PstartF: [evidence span withheld — anchor status: not_found]
- [other] Align each trimmed read to the reference template with a local pair-wise alignment and choose between sense and antisense templates based on the longest alignment.: [evidence span withheld — anchor status: not_found]
- [methods] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
- [readme] the analysis may be aborted with the error message: "Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.": "or the analysis may be aborted with the error message: “Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.”"
