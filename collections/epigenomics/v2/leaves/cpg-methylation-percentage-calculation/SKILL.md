---
name: cpg-methylation-percentage-calculation
description: Use when raw .ab1 Sanger trace files from direct-BSP (or cloning-BSP)
  experiments must be converted into per-CpG methylation percentages anchored to a
  reference .fasta sequence.
when_to_use_negative:
- Input is already a methylation matrix or table of methylation percentages — this
  skill operates on raw .ab1 chromatogram traces
- Data are high-throughput bisulfite sequencing reads (e.g. FASTQ/BS-seq); ABSP is
  designed for Sanger-style BSP trace files
- No CpG sites are covered by the sequencing reads after trimming — the analysis is
  designed to stop in this case
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: ABSP
  role: R tool that trims .ab1 traces, aligns reads to the reference, computes peakC/(peakC+peakT)
    or peakG/(peakG+peakA) methylation percentages per CpG, and writes methylation
    tables and plots
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which ABSP and its analysis Rmd scripts (ABSP_individual_analysis.Rmd,
    ABSP_functions.R) are executed
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: Shiny
  role: User-facing interface exposing the Individual analysis tab where .fasta and
    .ab1 inputs are supplied and the analysis is launched
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:079c791c092aadd53b155a20d4e2081dd88f91aa48ad25006d4443764657f5d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cpg-methylation-percentage-calculation

## Summary

Compute per-CpG methylation percentages from bisulfite sequencing PCR (BSP) chromatogram traces using the ABSP R tool, which converts dye-signal peak heights into methylation levels and maps them to reference CpG coordinates.

## When to use

Use when raw .ab1 Sanger trace files from direct-BSP (or cloning-BSP) experiments must be converted into per-CpG methylation percentages anchored to a reference .fasta sequence.

## When NOT to use

- Input is already a methylation matrix or table of methylation percentages — this skill operates on raw .ab1 chromatogram traces
- Data are high-throughput bisulfite sequencing reads (e.g. FASTQ/BS-seq); ABSP is designed for Sanger-style BSP trace files
- No CpG sites are covered by the sequencing reads after trimming — the analysis is designed to stop in this case

## Inputs

- Raw sequencing trace files in .ab1 format (Sanger chromatograms from bisulfite PCR)
- Reference sequence in .fasta format (with coordinates in the FASTA header, e.g. 'chr1:1-200')
- Optional reference genome assembly (e.g. GCF_900626175.2_cs10 .fna.gz) for genomic plots only

## Outputs

- Per-CpG methylation table (meth_table) with one percentage (0-100) per covered CpG site in the results folder
- Methylation table image (e.g. CDH1_High methylated DNA_3_meth_table.png)
- HTML analysis report in the reports folder

## How to apply

Load the deposited .ab1 sequencing traces and the reference sequence in .fasta format into ABSP (R, via the Shiny app or ABSP_individual_analysis.Rmd). Trim low-quality ends of each trace using combined trimming: Phred quality-score-based trimming with th_quality_error = 0.001 (99.9% base-calling accuracy, Q30) and primary-peak-ratio-based trimming with th_mixed_position = 0.75 and th_mixed_perc = 75%. Align each trimmed read to the reference with a local pair-wise alignment, choosing sense or antisense template by longest alignment, and map cytosine positions onto the read via the offset 'template position − Sstart + Pstart' (e.g. 100 − 30 + 22 = 92). For each covered CpG, compute methylation as peakC/(peakC+peakT)×100 on forward reads and peakG/(peakG+peakA)×100 on reverse reads from the chromatogram peak heights. Write the per-CpG methylation table to the results folder and render it as the methylation table image (e.g. 'CDH1_High methylated DNA_3_meth_table.png'); if no CpG sites are covered, stop with the error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'

## Related tools

- **ABSP** (R tool that trims .ab1 traces, aligns reads to the reference, computes peakC/(peakC+peakT) or peakG/(peakG+peakA) methylation percentages per CpG, and writes methylation tables and plots) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which ABSP and its analysis Rmd scripts (ABSP_individual_analysis.Rmd, ABSP_functions.R) are executed) — https://github.com/ABSP-methylation-tool/ABSP
- **Shiny** (User-facing interface exposing the Individual analysis tab where .fasta and .ab1 inputs are supplied and the analysis is launched) — https://github.com/ABSP-methylation-tool/ABSP

## Examples

```
Rscript ABSP_individual_analysis.Rmd  # or open app.R in RStudio and click 'Run App', then use the Individual analysis tab with .fasta reference and .ab1 traces
```

## Evaluation signals

- A meth_table output file exists in the results folder and contains one methylation percentage per covered CpG site
- All computed methylation percentages fall within the 0-100 range
- For the deposited CDH1 high methylated DNA example, the meth_table reproduces the ABSP example output 'CDH1_High methylated DNA_3_meth_table.png'
- Cytosine genomic coordinates equal template position − Sstart + Pstart from the local pair-wise alignment (offset arithmetic check)
- Trimmed sequences are neither over-trimmed nor retain low-quality ends, as judged from the 'Trimming plot' combining quality-score (orange) and primary-ratio (blue) results into the final green sequence

## Limitations

- If the Phred-quality trimming is too aggressive or primary-peak-ratio trimming too strict, sequences may become too short with few covered CpG sites, or the analysis aborts with 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- The reference genome is only needed for genomic visualization; with an arbitrary genome the displayed coordinates must not be taken into consideration in tables or plots
- For custom reporter sequences absent from reference genomes, arbitrary coordinates (e.g. 'chr1:1-200' for a 200 bp sequence) must be set in the FASTA header

## Evidence

- [full_text] ABSP computes each CpG methylation percentage from dye-signal peak heights as peakC/(peakC+peakT)×100 for forward sequencing reads and peakG/(peakG+peakA)×100 for reverse sequencing reads.: [evidence span withheld — anchor status: not_found]
- [methods] ABSP maps cytosine positions from the template DNA onto the sequencing read using the local pair-wise alignment offset.: [evidence span withheld — anchor status: not_found]
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages"
- [readme] The README FAQ explains combined trimming and how to adjust thresholds when sequences become too short.: "The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green)"
- [methods] The trimming threshold th_quality_error defaults to 0.001, corresponding to 99.9% base-calling accuracy and Phred quality score of 30.: "By default, this is set to 0.001, which corresponds to a base-calling accuracy of 99.9% and a Phred quality score of 30"
