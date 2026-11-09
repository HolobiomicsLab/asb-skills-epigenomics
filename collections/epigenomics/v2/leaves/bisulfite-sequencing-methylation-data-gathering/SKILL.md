---
name: bisulfite-sequencing-methylation-data-gathering
description: Use when per-sample individual ABSP methylation tables already exist
  for multiple .ab1-derived samples assigned to two groups (e.g. high vs low methylated
  DNA) and group-level CpG methylation comparison, plots, and significance testing
  are needed.
when_to_use_negative:
- Input is raw .ab1 trace files and no individual sample analysis has been run yet
  — the Grouped analysis requires individually analyzed samples
- Only a single sample is available — group comparison statistics require at least
  two groups of samples
- You need genomic-coordinate visualization for a sequence not in a reference genome
  — use the arbitrary-genome FAQ workflow instead of expecting correct coordinates
  in grouped plots/tables
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: ABSP
  role: Provides the 'Grouped analysis' tab that gathers individual-sample methylation
    tables, runs two-by-two Student's t-tests, and generates boxplots, lollipop plots,
    and methylation profile plots
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which the ABSP grouped analysis Rmd workflow (ABSP_grouped_analysis.Rmd)
    executes
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: RStudio
  role: IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the Shiny
    app via the 'Run App' button
- name: shiny
  role: User-friendly interface of ABSP used to configure grouped-analysis inputs,
    group assignments, and plotting parameters
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:baedceda2dd070d27a1f510a4364c5a04f17566aea40b170d12f04638e57b388
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  - shiny
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# bisulfite-sequencing-methylation-data-gathering

## Summary

Aggregate individually analyzed BSP samples into group comparisons with ABSP's Shiny 'Grouped analysis' tab, computing two-by-two Student's t-tests on per-CpG methylation and generating boxplots, lollipop plots, and significance-annotated methylation profile plots. It converts per-sample methylation tables into group-level statistics and publication-ready graphics.

## When to use

Use when per-sample individual ABSP methylation tables already exist for multiple .ab1-derived samples assigned to two groups (e.g. high vs low methylated DNA) and group-level CpG methylation comparison, plots, and significance testing are needed.

## When NOT to use

- Input is raw .ab1 trace files and no individual sample analysis has been run yet — the Grouped analysis requires individually analyzed samples
- Only a single sample is available — group comparison statistics require at least two groups of samples
- You need genomic-coordinate visualization for a sequence not in a reference genome — use the arbitrary-genome FAQ workflow instead of expecting correct coordinates in grouped plots/tables

## Inputs

- Individual-sample methylation tables (e.g. *_meth_table outputs) produced by ABSP individual analysis from .ab1 trace files
- Sample-to-group assignments (e.g. high-methylated vs low-methylated collections)
- Plotting parameters: CpG position labels, as-is vs proportional scaling
- For cloning-BSP: per-clone methylation percentages for status conversion

## Outputs

- Per-CpG and mean-methylation boxplots (PNG)
- Lollipop plots of per-replicate methylation states (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png')
- Methylation profile plot with significance annotation (e.g. 'Test sequence_meth_profile_collection1_proportional_psign.png')
- Grouped .html report in the 'reports' folder
- methylation_status_table.csv for cloning-BSP grouped analysis

## How to apply

In RStudio, launch the ABSP Shiny app (open app.R, click 'Run App'), then load the per-sample individual methylation tables (e.g. *_meth_table outputs such as 'CDH1_High methylated DNA_3_meth_table.png' inputs from the 'examples' folder) into the 'Grouped analysis' tab. Assign samples to high- and low-methylated collections and set plotting parameters (CpG position labels such as 'CpG numbers' or 'None' for arbitrary genomes, proportional/as-is scaling). Run the analysis, which gathers samples, performs two-by-two Student's t-tests comparing CpG methylation between groups (e.g. mean 1.43% ±1.87% vs 95.54% ±0.67%, P = 2.34e-05 in the example), and generates per-CpG and mean-methylation boxplots, per-replicate lollipop plots, and methylation profile plots annotated with significance for every CpG covered. For cloning-BSP data, statuses are assigned by fixed thresholds: 0–20% unmethylated, 80–100% methylated, 20–80% annotated as not available, and clones with >20% partially methylated CpGs flagged as potential clone mixes with all data annotated NA. Outputs are written to the 'results' folder with the .html report to the 'reports' folder.

## Related tools

- **ABSP** (Provides the 'Grouped analysis' tab that gathers individual-sample methylation tables, runs two-by-two Student's t-tests, and generates boxplots, lollipop plots, and methylation profile plots) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which the ABSP grouped analysis Rmd workflow (ABSP_grouped_analysis.Rmd) executes) — https://github.com/ABSP-methylation-tool/ABSP
- **RStudio** (IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the Shiny app via the 'Run App' button)
- **shiny** (User-friendly interface of ABSP used to configure grouped-analysis inputs, group assignments, and plotting parameters)

## Examples

```
rstudio: open 'ABSP Rproject.Rproject' and 'app.R', click 'Run App'; equivalently: shiny::runApp("app.R")
```

## Evaluation signals

- The grouped-results directory contains the lollipop plot PNG (e.g. *_lollipop_replicates_as-is_proportional.png)
- The methylation profile PNG carries the '_psign' significance suffix and significance marks are present for every CpG site covered by the sequencing results
- The grouped .html report exists in the 'reports' folder and all plots/tables are in the 'results' folder
- Comparative statistics report a Student's t-test P-value (example: 2.34e-05) with group means matching the expected methylation contrast (e.g. 1.43% vs 95.54%)
- For cloning-BSP, methylation_status_table.csv exists with statuses restricted to unmethylated/methylated/NA and clones with >20% partial CpGs flagged as potential clone mixes

## Limitations

- If an arbitrary reference genome is used, coordinates must not be taken into consideration in tables or plots; 'CpG numbers' or 'None' options are recommended for CpG position labels in Grouped Analysis
- The reference genome is only necessary for visualization of the reference sequence in genomic plots and is not needed for computing methylation percentages
- On Linux, initial package installation during first ABSP launch may take considerable time; clones with >20% partially methylated CpGs are flagged as potential clone mixes and their data annotated as not available

## Evidence

- [readme] After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting.: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [readme] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [readme] the reference genome is only necessary for visualization of the reference sequence in genomic plots: "First, the reference genome is only necessary for visualization of the reference sequence in genomic plots but it is not needed for computing methylation percentages."
