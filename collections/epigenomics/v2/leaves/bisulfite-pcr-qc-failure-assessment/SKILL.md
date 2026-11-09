---
name: bisulfite-pcr-qc-failure-assessment
description: Use when ABSP direct-BSP analysis of .ab1 reads yields a very short trimmed
  sequence covering few CpG sites, or aborts with 'no CpG sites were found covered
  by sequencing results', and the trimming thresholds must be diagnosed and adjusted.
when_to_use_negative:
- The problem is a missing or unmatched reference genome — ABSP computes methylation
  percentages without a reference genome, which is only needed for genomic visualization,
  so threshold tuning will not fix plot/coordinate issues
- Input data is already trimmed FASTQ or a precomputed methylation table rather than
  raw .ab1 chromatogram traces, since ABSP's trimming/QC workflow operates on trace
  files
- The trimmed sequence is of adequate length and covers the expected CpG sites — in
  that case proceed directly to methylation computation and grouped analysis instead
  of relaxing QC thresholds
tools:
- name: ABSP
  role: R Shiny tool whose 'Individual analysis' tab and Trimming plot are used to
    detect and diagnose trimming/QC failures on .ab1 reads
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime in which ABSP executes and whose Rmd script (ABSP_individual_analysis.Rmd,
    lines 607/612/614) exposes the trimming thresholds
  repo: https://www.r-project.org/
- name: RStudio
  role: IDE used to open the ABSP Rproject and app.R and edit the trimming thresholds
    in the Rmd scripts
  repo: https://www.rstudio.com/products/rstudio/download/
- name: shiny
  role: Package powering the ABSP user interface (Individual analysis / Sequencing
    trimming tabs) installed on first app launch
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:18ccfc56533b15c4a32b5804ee00942d7e347ae2c62647a8a17df9e79f1c730c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - Gviz
  - R
  - RStudio
  - shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bisulfite-pcr-qc-failure-assessment

## Summary

A diagnostic skill for interpreting sequencing-trimming and quality-control failures in ABSP's analysis of direct-BSP .ab1 chromatogram traces, where too-aggressive Phred quality-score trimming or too-strict primary peak-ratio thresholds produce short trimmed sequences or aborted analyses with no covered CpG sites.

## When to use

Use when ABSP direct-BSP analysis of .ab1 reads yields a very short trimmed sequence covering few CpG sites, or aborts with 'no CpG sites were found covered by sequencing results', and the trimming thresholds must be diagnosed and adjusted.

## When NOT to use

- The problem is a missing or unmatched reference genome — ABSP computes methylation percentages without a reference genome, which is only needed for genomic visualization, so threshold tuning will not fix plot/coordinate issues
- Input data is already trimmed FASTQ or a precomputed methylation table rather than raw .ab1 chromatogram traces, since ABSP's trimming/QC workflow operates on trace files
- The trimmed sequence is of adequate length and covers the expected CpG sites — in that case proceed directly to methylation computation and grouped analysis instead of relaxing QC thresholds

## Inputs

- Raw bisulfite sequencing trace files in .ab1 format
- Reference sequence of the studied region (e.g. CDH1 promoter) in .fasta format
- ABSP_individual_analysis.Rmd script with trimming parameters th_quality_error (line 607), th_mixed_position (line 612), th_mixed_perc (line 614)

## Outputs

- Trimming plot showing quality-score (orange), primary-ratio (blue) and final trimmed (green) sequence extents
- Per-sample CpG methylation table in the 'Methylation' / 'Raw Sequence' > 'Table' tabs
- .html QC/analysis report saved in the 'reports' folder and output files in the 'results' folder

## How to apply

First inspect the 'Trimming plot' in the 'Sequencing trimming' tab of the ABSP Shiny app, where results of both trimming methods — Phred quality scores (orange) and primary peak ratios (blue) — are combined into the final trimmed sequence (green); the goal is that low-quality ends are removed without over-trimming. If the quality-score method is too aggressive, lower the minimum base-calling error probability threshold th_quality_error in ABSP_individual_analysis.Rmd (line 607) from the default 0.001 (99.9% base-calling accuracy, Phred 30) to e.g. 0.01 (99% accuracy, Phred 20). If the primary-ratio method is too strict, relax th_mixed_position (default 0.75, line 612, the minimum primary peak ratio for a position to be non-mixed) and/or th_mixed_perc (default 75%, line 614, the minimum percentage of non-mixed positions in the trimmed sequence). Re-run the individual analysis and check the 'Raw Sequence' > 'Table' tab to confirm CpG coverage before proceeding to grouped analysis; on the EpigenDx CDH1 example, note that 5/6 forward reads legitimately fail trimming/QC, so some read loss is expected.

## Related tools

- **ABSP** (R Shiny tool whose 'Individual analysis' tab and Trimming plot are used to detect and diagnose trimming/QC failures on .ab1 reads) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime in which ABSP executes and whose Rmd script (ABSP_individual_analysis.Rmd, lines 607/612/614) exposes the trimming thresholds) — https://www.r-project.org/
- **RStudio** (IDE used to open the ABSP Rproject and app.R and edit the trimming thresholds in the Rmd scripts) — https://www.rstudio.com/products/rstudio/download/
- **shiny** (Package powering the ABSP user interface (Individual analysis / Sequencing trimming tabs) installed on first app launch) — https://github.com/ABSP-methylation-tool/ABSP

## Evaluation signals

- On the EpigenDx CDH1 example, the expected QC outcome is that 5/6 forward reads fail trimming/QC — deviation in either direction warrants investigation
- The final (green) trimmed sequence on the Trimming plot must retain enough CpG sites for the 'Methylation' tab to be populated, and the analysis must not abort with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results.'
- After threshold adjustment, reproduced grouped statistics should match CpG site #4 methylation 0.47% ± 0.81% (low) vs 81.44% ± 3.96% (high) with P = 0.00051, and mean methylation 1.43% ± 1.87% vs 95.54% ± 0.67% with P = 2.34e−05
- Verify that th_quality_error remains a base-calling error probability consistent with intent: 0.001 ≙ 99.9% accuracy / Phred 30, 0.01 ≙ 99% / Phred 20
- Check that edits to thresholds were made at the documented script locations (ABSP_individual_analysis.Rmd lines 607, 612, 614) and that the .html report was regenerated in the 'reports' folder

## Limitations

- Lowering th_quality_error trades base-calling stringency for sequence length: going from 0.001 (Phred 30) to 0.01 (Phred 20) accepts 99% rather than 99.9% base-calling accuracy, so low-quality ends may be retained downstream
- Relaxing th_mixed_position/th_mixed_perc risks treating genuinely mixed (heterozygously methylated) positions as non-mixed, biasing per-CpG methylation estimates
- Some reads may legitimately fail QC even on control data (5/6 forward reads on the CDH1 example), so read loss alone is not proof of a misconfigured threshold
- On Linux machines, the package installation during the first ABSP launch may take a considerable amount of time, delaying iteration on QC fixes

## Evidence

- [readme] First, check the “Trimming plot” in the “Sequencing trimming” tab, as this will provide insight into the trimming step results.: "First, check the “Trimming plot” in the “Sequencing trimming” tab, as this will provide insight into the trimming step results."
- [readme] The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green).: "The results from both trimming methods—based on quality scores (orange) and primary ratios (blue)—are combined to produce the final trimmed sequence (green)."
- [methods] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
- [readme] By default, this is set to 0.001, which corresponds to a base-calling accuracy of 99.9% and a Phred quality score of 30. Lowering it to 0.01, for example, corresponds to a base-calling accuracy of 99% and a Phred quality score of 20.: "By default, this is set to 0.001, which corresponds to a base-calling accuracy of 99.9% and a Phred quality score of 30. Lowering it to 0.01, for ex"
- [methods] `th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be considered non-mixed: "`th_mixed_position` (default: 0.75, line 612 in the ABSP_individual_analysis.Rmd file), this is the minimum primary peak ratio for a position to be"
