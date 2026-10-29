---
name: shiny-app-grouped-analysis-configuration
description: Use when individually analyzed BSP samples (per-sample methylation tables
  from direct-BSP or cloning-BSP .ab1 trace files) must be gathered into groups, compared
  statistically, and visualized with publication-ready boxplots, lollipop plots, and
  methylation profile plots.
when_to_use_negative:
- Individual samples have not yet been processed by the ABSP 'Individual analysis'
  tab — grouped analysis gathers pre-computed individual methylation tables, not raw
  .ab1 trace files
- The goal is a single sample's CpG methylation table or trimming diagnostics (use
  the 'Individual analysis' tab with its Sequencing trimming and Methylation tabs
  instead)
- Only an arbitrary reference genome is available and per-CpG genomic coordinates
  matter — use 'CpG numbers' or 'None' labels and disregard coordinates in tables/plots,
  or handle coordinates separately
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0092
tools:
- name: ABSP
  role: R tool providing the Shiny app whose 'Grouped analysis' tab gathers samples,
    generates lollipop plots, and runs comparative statistics on CpG methylation
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: Shiny
  role: Built-in R Shiny app framework delivering the user interface for the 'Grouped
    analysis' tab
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: RStudio
  role: IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and click 'Run App'
    to launch the ABSP Shiny app
- name: R
  role: Language in which ABSP is implemented and in which the grouped analysis and
    Student's t-tests are computed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:0c182e141d145f8816be581dd003faa6788633856ed3948a77ca96f546dceb23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  - Shiny
  - shiny
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# shiny-app-grouped-analysis-configuration

## Summary

Configure and run the 'Grouped analysis' tab of ABSP's R Shiny app to gather individually analyzed bisulfite sequencing PCR (BSP) samples into collections, generate lollipop and methylation-profile plots, and run two-by-two Student's t-tests comparing CpG methylation between groups.

## When to use

Use when individually analyzed BSP samples (per-sample methylation tables from direct-BSP or cloning-BSP .ab1 trace files) must be gathered into groups, compared statistically, and visualized with publication-ready boxplots, lollipop plots, and methylation profile plots.

## When NOT to use

- Individual samples have not yet been processed by the ABSP 'Individual analysis' tab — grouped analysis gathers pre-computed individual methylation tables, not raw .ab1 trace files
- The goal is a single sample's CpG methylation table or trimming diagnostics (use the 'Individual analysis' tab with its Sequencing trimming and Methylation tabs instead)
- Only an arbitrary reference genome is available and per-CpG genomic coordinates matter — use 'CpG numbers' or 'None' labels and disregard coordinates in tables/plots, or handle coordinates separately

## Inputs

- Per-sample methylation tables from completed ABSP individual analyses (e.g. individual_results_direct tables for CDH1 high/low methylated DNA samples)
- Grouping assignments of samples into collections (e.g. high-methylated vs low-methylated DNA)
- Plotting parameter selections: CpG position labels, proportional/as-is scaling

## Outputs

- Lollipop plots of per-replicate methylation states (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png')
- Per-CpG and mean-methylation boxplots
- Methylation profile plot with significance annotation for all covered CpG sites (e.g. 'Test sequence_meth_profile_collection1_proportional_psign.png')
- Grouped .html report written to the 'reports' folder
- Output files written to the 'results' folder

## How to apply

First complete the individual analysis for every sample so that per-sample methylation tables exist (e.g. CDH1 individual_results_direct outputs such as 'CDH1_High methylated DNA_3_meth_table.png'). Open 'ABSP Rproject.Rproject' and 'app.R' in RStudio, click 'Run App', and navigate to the 'Grouped analysis' tab; assign samples to collections (e.g. high- versus low-methylated DNA) and select plotting parameters such as CpG position labels ('CpG numbers' or 'None' if using an arbitrary reference genome) and proportional/as-is scaling. Run the analysis, which gathers the samples and computes two-by-two Student's t-tests comparing CpG methylation between groups; in the example analysis this yielded 1.43% (±1.87%) mean methylation for low-methylated versus 95.54% (±0.67%) for high-methylated DNA with P = 2.34e−05. Verify that the grouped-results directory contains the lollipop plot PNG, the methylation profile PNG with the '_psign' significance suffix, and the grouped .html report in the 'reports' folder.

## Related tools

- **ABSP** (R tool providing the Shiny app whose 'Grouped analysis' tab gathers samples, generates lollipop plots, and runs comparative statistics on CpG methylation) — https://github.com/ABSP-methylation-tool/ABSP
- **Shiny** (Built-in R Shiny app framework delivering the user interface for the 'Grouped analysis' tab) — https://github.com/ABSP-methylation-tool/ABSP
- **RStudio** (IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and click 'Run App' to launch the ABSP Shiny app)
- **R** (Language in which ABSP is implemented and in which the grouped analysis and Student's t-tests are computed)

## Examples

```
shiny::runApp("app.R")  # open ABSP Rproject.Rproject in RStudio, launch the Shiny app, then use the 'Grouped analysis' tab
```

## Evaluation signals

- The grouped-results directory contains the lollipop plot PNG (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png')
- The methylation profile PNG carries the '_psign' significance suffix and shows significance marks for every CpG site covered by the sequencing results
- A grouped .html report is present in the 'reports' folder and output files appear in the 'results' folder
- Two-by-two Student's t-test P-values are reported for each CpG comparison; in the example, high vs low methylated DNA differ at P = 2.34e−05 with means of 95.54% (±0.67%) vs 1.43% (±1.87%)
- Samples are correctly assigned to the intended collections before running the analysis

## Limitations

- Grouped analysis requires that all samples were first individually analyzed; it cannot process raw .ab1 trace files directly
- With an arbitrary reference genome, coordinates displayed in plots/tables must not be taken into consideration; choose 'CpG numbers' or 'None' for CpG position labels in the Grouped Analysis
- If no CpG sites are covered by sequencing results after trimming, the analysis stops with the error 'Analysis has been stopped as no CpG sites were found covered by sequencing results.'

## Evidence

- [readme] After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting.: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [methods] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [results] a Student's t-test P-value of 2.34e−05, confirming the methylation difference between groups: [evidence span withheld — anchor status: not_found]
- [readme] it is recommended to choose either the "CpG numbers" or "None" options for CpG position labels in the Grouped Analysis.: "it is recommended to choose either the "CpG numbers" or "None" options for CpG position labels in the Grouped Analysis."
