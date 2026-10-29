---
name: read-to-reference-alignment-for-methylation-analysis
description: Use when chromatogram trace reads (.ab1) from direct-BSP or cloning-BSP
  must be aligned to a reference .fasta to identify non-CpG C/G positions for bisulfite
  conversion-rate QC and CpG methylation computation.
when_to_use_negative:
- Input is FASTQ/NGS reads rather than Sanger .ab1 chromatogram traces — ABSP is built
  for raw sequencing trace files
- Only the reference genome for genomic visualization is needed — the README notes
  the reference genome is 'only necessary for visualization ... it is not needed for
  computing methylation percentages', which relies on the per-read reference .fasta
  alignment
- Data are already analyzed group-level methylation tables rather than individual
  trace files needing alignment and QC
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0622
tools:
- name: ABSP
  role: R tool whose ABSP_individual_analysis.Rmd routines parse .ab1 traces, align
    trimmed reads to the reference .fasta, and compute bisulfite conversion rates
    at non-CpG positions
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment for ABSP's trace parsing, alignment, and conversion-rate
    computation
- name: Shiny
  role: User-facing interface ('Individual analysis' tab) where the reference .fasta
    and .ab1 files are supplied to run the alignment and analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:8062641c66797d9d73832728b894ec4fbd0eaa04d1601473a5d5ba64831076e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# read-to-reference-alignment-for-methylation-analysis

## Summary

Align trimmed .ab1 sequencing reads to a reference .fasta sequence in ABSP to locate non-CpG cytosines (forward) and guanines (reverse), enabling per-position bisulfite conversion-rate computation and QC of aligned bisulfite sequencing PCR results.

## When to use

Use when chromatogram trace reads (.ab1) from direct-BSP or cloning-BSP must be aligned to a reference .fasta to identify non-CpG C/G positions for bisulfite conversion-rate QC and CpG methylation computation.

## When NOT to use

- Input is FASTQ/NGS reads rather than Sanger .ab1 chromatogram traces — ABSP is built for raw sequencing trace files
- Only the reference genome for genomic visualization is needed — the README notes the reference genome is 'only necessary for visualization ... it is not needed for computing methylation percentages', which relies on the per-read reference .fasta alignment
- Data are already analyzed group-level methylation tables rather than individual trace files needing alignment and QC

## Inputs

- .ab1 sequencing trace files (primary peak heights per position)
- Reference sequence in .fasta format
- ABSP_individual_analysis.Rmd parameters (th_quality_error, th_mixed_position, th_mixed_perc)

## Outputs

- Per-read alignment against the reference with identified non-CpG C (forward) and G (reverse) positions
- Per-read bisulfite conversion rate (numeric 0–1) with strand and QC flag (pass/fail at threshold 0.9)
- conversion_rate_qc.csv with one row per processed read
- CpG methylation levels computed on the aligned, trimmed sequence

## How to apply

Load the .ab1 trace files and matching reference .fasta (e.g. from the ABSP 'examples' folder) into R and use ABSP's trace-parsing routines in ABSP_individual_analysis.Rmd to extract primary peak heights per position. Trim low-quality ends first using the combined Phred-quality method (th_quality_error, default 0.001 ≈ Phred 30) and primary-peak-ratio method (th_mixed_position default 0.75, th_mixed_perc default 75%), since trimming results determine which region is aligned. Align each trimmed read to the reference sequence, then identify non-CpG cytosine positions on forward-aligned reads and non-CpG guanine positions on reverse reads (reverse-complement of forward / reverse reads). At each non-CpG position compute the conversion ratio from primary peak heights — peakT/(peakC+peakT) on the forward strand and peakA/(peakG+peakA) on the reverse strand — and average per-position ratios into one conversion rate per read. Reads whose average conversion rate falls below the default acceptance threshold of 0.9 fail quality control, so alignment must place non-CpG positions correctly before methylation percentages at CpG sites are computed.

## Related tools

- **ABSP** (R tool whose ABSP_individual_analysis.Rmd routines parse .ab1 traces, align trimmed reads to the reference .fasta, and compute bisulfite conversion rates at non-CpG positions) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment for ABSP's trace parsing, alignment, and conversion-rate computation)
- **Shiny** (User-facing interface ('Individual analysis' tab) where the reference .fasta and .ab1 files are supplied to run the alignment and analysis)

## Examples

```
Rscript -e "rmarkdown::render('ABSP_individual_analysis.Rmd', params=list(ref_fasta='examples/CDH1.fasta', ab1_files=list.files('examples','.ab1$', full.names=TRUE)))"
```

## Evaluation signals

- Each per-read conversion rate is a numeric value between 0 and 1
- Reads with conversion rate at or above 0.9 are labeled 'pass'; reads below 0.9 are flagged as failing QC
- conversion_rate_qc.csv contains exactly one row per processed example read, with per-read strand and QC flag
- Non-CpG C positions on forward reads use peakT/(peakC+peakT); reverse reads use peakA/(peakG+peakA) — verify per-position ratios follow the strand-specific formula
- Trimmed sequences must retain sufficient CpG coverage; abort condition 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results' indicates mis-trimming/mis-alignment

## Limitations

- Overly aggressive Phred-quality trimming or overly strict primary-peak-ratio trimming can leave sequences too short, reducing CpG coverage or aborting the analysis; thresholds th_quality_error (0.001), th_mixed_position (0.75), and th_mixed_perc (75%) may need adjustment in ABSP_individual_analysis.Rmd
- Reference genome files are only for genomic plot visualization; computing methylation percentages relies on the provided reference sequence .fasta, and arbitrary genomes change displayed coordinates
- Custom reporter sequences require arbitrary FASTA header coordinates (e.g. 'chr1:1-200') matching sequence length

## Evidence

- [full_text] ABSP computes the bisulfite conversion rate for each cytosine position outside CpG sites, and the average rate on the sequence must be higher than a provided threshold, which is 0.9 by default.: [evidence span withheld — anchor status: not_found]
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages"
- [readme] Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format.: "add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
- [methods] `th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed: "this is the minimum primary peak ratio for a position to be considered non-mixed"
- [readme] the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages.: "the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages"
