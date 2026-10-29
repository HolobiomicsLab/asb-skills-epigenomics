---
name: cpg-methylation-lollipop-plot-generation
description: Use when individual direct-BSP or cloning-BSP samples have already been
  analyzed in ABSP and you need per-replicate, per-CpG methylation-state visualizations
  comparing sample groups (e.g. high- vs low-methylated DNA).
when_to_use_negative:
- Individual samples have not yet been analyzed — lollipop plots require completed
  per-sample methylation tables from the 'Individual analysis' tab; run that first
- You only need CpG methylation percentages without visualization — the reference
  genome is not needed for computing methylation percentages, only for genomic plots,
  so skip plotting steps if statistics alone suffice
edam_operation: http://edamontology.org/operation_3200
tools:
- name: ABSP
  role: R-based tool whose Shiny 'Grouped analysis' tab gathers individually analyzed
    samples and generates the lollipop plots of methylation states
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment in which the ABSP Shiny app and its grouped-analysis scripts
    (ABSP_grouped_analysis.Rmd) execute
- name: RStudio
  role: IDE used to open 'ABSP Rproject.Rproject' and launch 'app.R' via the 'Run
    App' button
- name: shiny
  role: Provides the user-friendly graphical interface (Grouped analysis tab) through
    which plotting parameters are set and the analysis is launched
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:6b07ee024f57b37aee8c236469acdec582dd04fa6f0bb3fe1728533ea5409ac7
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
  edam_operation: http://edamontology.org/operation_3200
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# CpG methylation lollipop plot generation

## Summary

Generates per-replicate lollipop plots of CpG methylation states from ABSP's 'Grouped analysis' tab, converting individually analyzed bisulfite sequencing PCR (BSP) methylation tables into publication-ready visualization PNGs for each collection of samples.

## When to use

Use when individual direct-BSP or cloning-BSP samples have already been analyzed in ABSP and you need per-replicate, per-CpG methylation-state visualizations comparing sample groups (e.g. high- vs low-methylated DNA).

## When NOT to use

- Individual samples have not yet been analyzed — lollipop plots require completed per-sample methylation tables from the 'Individual analysis' tab; run that first
- You only need CpG methylation percentages without visualization — the reference genome is not needed for computing methylation percentages, only for genomic plots, so skip plotting steps if statistics alone suffice

## Inputs

- Individual-analysis methylation tables produced by ABSP (e.g. CDH1 individual_results_direct tables such as 'CDH1_High methylated DNA_3_meth_table.png') located in the ABSP 'examples' or 'results' folder
- Group assignments of samples into collections (e.g. high- vs low-methylated DNA groups)
- Grouped-analysis plotting parameters: CpG position label choice and as-is/proportional scaling

## Outputs

- Per-replicate lollipop plot PNGs (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png') written to the 'results' folder under grouped_results_direct_<timestamp>/lollipop_plots/plots_replicates/
- Grouped .html report written to the 'reports' folder

## How to apply

After running ABSP's 'Individual analysis' tab on raw .ab1 trace files against a .fasta reference, open the 'Grouped analysis' tab in the Shiny app, assign individually analyzed samples to their collections (e.g. high-methylated vs low-methylated), and select plotting parameters such as CpG position labels ('CpG numbers' or 'None' when no true reference genome is used) and as-is/proportional scaling. Running the analysis gathers the samples, generates lollipop plots of methylation states per replicate (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png'), and alongside boxplots and methylation profile plots writes the PNG outputs to the 'results' folder and a grouped .html report to the 'reports' folder. Rationale: the article emphasizes ABSP's automated workflow 'spanning from trace file sequencing results to data visualization and statistical analysis', so the lollipop step is executed together with two-by-two Student's t-tests comparing CpG methylation between groups.

## Related tools

- **ABSP** (R-based tool whose Shiny 'Grouped analysis' tab gathers individually analyzed samples and generates the lollipop plots of methylation states) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment in which the ABSP Shiny app and its grouped-analysis scripts (ABSP_grouped_analysis.Rmd) execute)
- **RStudio** (IDE used to open 'ABSP Rproject.Rproject' and launch 'app.R' via the 'Run App' button)
- **shiny** (Provides the user-friendly graphical interface (Grouped analysis tab) through which plotting parameters are set and the analysis is launched)

## Examples

```
Open 'ABSP Rproject.Rproject' in RStudio, open 'app.R', click 'Run App', then in the 'Grouped analysis' tab load individual methylation tables, assign high/low-methylated collections, and run to produce 'Test sequence_lollipop_replicates_as-is_proportional.png' in results/
```

## Evaluation signals

- The grouped-results directory contains the lollipop plot PNG (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png')
- The grouped .html report is present in the 'reports' folder
- Lollipop plots reflect the expected methylation contrast between groups (e.g. mean methylation 1.43% ± 1.87% low-methylated vs 95.54% ± 0.67% high-methylated, Student's t-test P = 2.34e−05)
- Filenames reflect the chosen plotting parameters (as-is vs proportional scaling; chosen CpG position label option)
- Every replicate in the collections appears as a lollipop track in the plot

## Limitations

- Lollipop plotting depends on prior individual analyses passing QC — samples whose reads fail trimming (in the example, 5/6 forward reads failed) may be under-represented
- If no true reference genome is used, CpG position coordinates in plots are arbitrary and should not be taken into consideration; use 'CpG numbers' or 'None' labels
- The lollipop step is coupled to ABSP's grouped analysis; plots are generated together with comparative statistics and profile plots, not as a standalone module

## Evidence

- [methods] Grouped samples analysis: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [readme] Lollipop output example: "generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [readme] Grouped analysis tab usage: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [methods] Individual analysis prerequisite: "Navigate to the "Individual analysis" tab and complete the entries and add input files such as the reference sequence in .fasta format and sequencing files in .ab1 format"
