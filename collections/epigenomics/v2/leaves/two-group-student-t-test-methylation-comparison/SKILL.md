---
name: two-group-student-t-test-methylation-comparison
description: Use when individually analyzed direct-BSP or cloning-BSP methylation
  tables exist for ≥2 sample groups and you need comparative statistics (Student's
  t-test P-values) plus grouped visualization of CpG methylation.
when_to_use_negative:
- Samples have not yet been individually analyzed — ABSP's grouped analysis requires
  per-sample methylation tables from the 'Individual analysis' tab as input
- More than two groups or complex designs are needed — ABSP's grouped analysis performs
  two-by-two Student's t-tests between collections, not multi-factor or non-parametric
  comparisons
- You only need methylation percentages without group comparison — the 'Individual
  analysis' tab alone computes CpG methylation levels per sample
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3676
- http://edamontology.org/topic_0654
tools:
- name: ABSP
  role: Automated R tool whose Shiny 'Grouped analysis' tab gathers samples, runs
    two-by-two Student's t-tests, and generates boxplots, lollipop plots, and significance-annotated
    methylation profile plots
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: Shiny
  role: Built-in R Shiny app providing the user-friendly 'Grouped analysis' interface
    used to assign samples to groups and select plotting parameters
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment for ABSP; grouped analysis scripts (ABSP_grouped_analysis.Rmd,
    ABSP_functions.R) execute in R
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: RStudio
  role: IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the ABSP
    app via the 'Run App' button
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:23f50ec1c917f267a51ab1e4135e24c311ab67b48fa14542ca06b26b1f59dfcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  - Shiny
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3676
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# two-group-student-t-test-methylation-comparison

## Summary

Apply ABSP's Shiny 'Grouped analysis' tab to gather individually analyzed bisulfite sequencing PCR (BSP) samples into two collections (e.g. high- vs low-methylated DNA) and compare CpG methylation between groups using two-by-two Student's t-tests, boxplots, lollipop plots, and significance-annotated methylation profile plots.

## When to use

Use when individually analyzed direct-BSP or cloning-BSP methylation tables exist for ≥2 sample groups and you need comparative statistics (Student's t-test P-values) plus grouped visualization of CpG methylation.

## When NOT to use

- Samples have not yet been individually analyzed — ABSP's grouped analysis requires per-sample methylation tables from the 'Individual analysis' tab as input
- More than two groups or complex designs are needed — ABSP's grouped analysis performs two-by-two Student's t-tests between collections, not multi-factor or non-parametric comparisons
- You only need methylation percentages without group comparison — the 'Individual analysis' tab alone computes CpG methylation levels per sample

## Inputs

- Individual-sample methylation tables produced by ABSP 'Individual analysis' (e.g. CDH1 individual_results_direct *_meth_table outputs from .ab1 trace files with a reference .fasta)
- Group assignment of samples into two collections (e.g. high-methylated vs low-methylated DNA) entered in the 'Grouped analysis' tab
- Plotting parameters: CpG position labels option (positions/CpG numbers/None) and proportional vs as-is scaling

## Outputs

- Per-CpG and mean-methylation boxplots comparing the two groups
- Lollipop plots of per-replicate methylation states (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png')
- Methylation profile plot with significance annotation for all covered CpG sites (e.g. 'Test sequence_meth_profile_collection1_proportional_psign.png' with '_psign' suffix)
- Student's t-test P-values comparing CpG methylation between groups (e.g. P = 2.34e−05)
- Grouped .html report in the 'reports' folder and all plot outputs in the 'results' folder

## How to apply

First complete the individual analysis in ABSP (v1.2.2) so each sample has a *_meth_table output, e.g. the CDH1 example tables shipped in the 'examples' folder under individual_results_direct. In RStudio, open the ABSP Shiny app (app.R via 'Run App'), navigate to the 'Grouped analysis' tab, assign samples to the high- and low-methylated collections, and select plotting parameters (CpG position labels such as 'CpG numbers' or 'None' when using an arbitrary reference genome; proportional/as-is scaling). Run the grouped analysis, which gathers samples, computes two-by-two Student's t-tests comparing CpG methylation between groups, and generates per-CpG and mean-methylation boxplots, per-replicate lollipop plots, and methylation profile plots annotated with significance for all covered CpG sites. In the published example this yielded a mean CpG methylation rate of 1.43% (±1.87%) in low-methylated DNA versus 95.54% (±0.67%) in high-methylated DNA with Student's t-test P = 2.34e−05. All plots are written to the 'results' folder and the grouped .html report to the 'reports' folder.

## Related tools

- **ABSP** (Automated R tool whose Shiny 'Grouped analysis' tab gathers samples, runs two-by-two Student's t-tests, and generates boxplots, lollipop plots, and significance-annotated methylation profile plots) — https://github.com/ABSP-methylation-tool/ABSP
- **Shiny** (Built-in R Shiny app providing the user-friendly 'Grouped analysis' interface used to assign samples to groups and select plotting parameters) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment for ABSP; grouped analysis scripts (ABSP_grouped_analysis.Rmd, ABSP_functions.R) execute in R) — https://github.com/ABSP-methylation-tool/ABSP
- **RStudio** (IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the ABSP app via the 'Run App' button)

## Examples

```
RStudio: open 'ABSP Rproject.Rproject' and 'app.R', then click 'Run App' and use the 'Grouped analysis' tab (ABSP v1.2.2)
```

## Evaluation signals

- The grouped-results directory contains the lollipop plot PNG (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png')
- The methylation profile PNG carries the '_psign' significance suffix (e.g. 'Test sequence_meth_profile_collection1_proportional_psign.png')
- Significance marks are present for every CpG site covered by the sequencing results in the methylation profile plot
- The grouped .html report exists in the 'reports' folder and reports a Student's t-test P-value; the example benchmark is 1.43% (±1.87%) vs 95.54% (±0.67%) methylation with P = 2.34e−05
- Group mean methylation rates fall near the expected extremes for the example comparison (low-methylated ~1.43%, high-methylated ~95.54%)

## Limitations

- The reference genome is only necessary for visualization of the reference sequence in genomic plots; with an arbitrary genome, coordinates in tables/plots must be ignored and 'CpG numbers' or 'None' CpG position labels are recommended
- If trimming leaves too few CpG sites, the analysis aborts with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results'; thresholds th_quality_error (default 0.001) and th_mixed_position (default 0.75) in ABSP_individual_analysis.Rmd may need adjustment
- Grouped comparison is limited to two-by-two Student's t-tests between assigned collections; the 'Multiple analyses' tab is needed for multiple samples and/or multiple grouping analyses

## Evidence

- [readme] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [results] In the example analysis, ABSP's grouped analysis found a mean CpG methylation rate of 1.43% (±1.87%) in low-methylated DNA versus 95.54% (±0.67%) in high-methylated DNA, with a Student's t-test P-value of 2.34e−05: [evidence span withheld — anchor status: not_found]
- [readme] the reference genome is only necessary for visualization of the reference sequence in genomic plots: "First, the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages."
- [readme] After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting.: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [methods] the results must be validated by checking that the grouped-results directory contains the lollipop plot PNG and the methylation profile PNG with the '_psign' significance suffix: "the reference genome is only necessary for visualization of the reference sequence in genomic plots"
