---
name: methylation-profile-significance-plotting
description: Use when individually analyzed bisulfite sequencing PCR (.ab1 trace)
  samples must be gathered into groups (e.g. high- vs low-methylated DNA) and compared
  with t-tests plus per-CpG significance-annotated profile plots.
when_to_use_negative:
- Input is raw .ab1 trace files that have not yet been processed by an ABSP individual
  analysis — use the 'Individual analysis' tab first, since grouped analysis consumes
  individual methylation tables
- You only need per-sample CpG methylation percentages without group comparison or
  significance annotation — the individual analysis report suffices
- Multiple samples or multiple groupings must be launched in one click — the 'Multiple
  analyses' tab with input tables is the intended path instead
edam_operation: http://edamontology.org/operation_0337
tools:
- name: ABSP
  role: R/Shiny tool whose 'Grouped analysis' tab gathers samples, runs comparative
    statistics, and generates lollipop and significance-annotated methylation profile
    plots
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: Shiny
  role: Built-in R Shiny app providing the user-friendly 'Grouped analysis' tab interface
    with plotting parameter selection
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: RStudio
  role: Used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the ABSP app
    via the 'Run App' button
- name: R
  role: Underlying R environment in which ABSP's grouped analysis, t-tests, and plot
    generation execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:1b4daa2d61e32458e69bb009f81571f7ea58840f4c6587d378ea1bb6a99768fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  - Shiny
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# methylation-profile-significance-plotting

## Summary

In ABSP's Shiny 'Grouped analysis' tab, gathered per-sample CpG methylation tables from direct-BSP or cloning-BSP individual analyses are compared between collections with two-by-two Student's t-tests and rendered as publication-ready lollipop plots and methylation profile plots annotated with per-CpG significance ('_psign' outputs).

## When to use

Use when individually analyzed bisulfite sequencing PCR (.ab1 trace) samples must be gathered into groups (e.g. high- vs low-methylated DNA) and compared with t-tests plus per-CpG significance-annotated profile plots.

## When NOT to use

- Input is raw .ab1 trace files that have not yet been processed by an ABSP individual analysis — use the 'Individual analysis' tab first, since grouped analysis consumes individual methylation tables
- You only need per-sample CpG methylation percentages without group comparison or significance annotation — the individual analysis report suffices
- Multiple samples or multiple groupings must be launched in one click — the 'Multiple analyses' tab with input tables is the intended path instead

## Inputs

- Individual-sample methylation tables from completed ABSP individual analyses (e.g. *_meth_table PNG outputs under examples/.../individual_results_direct/tables)
- Group assignments of samples into collections (e.g. high-methylated vs low-methylated DNA)
- Plotting parameter selections (CpG position labels, proportional/as-is scaling)

## Outputs

- Methylation profile plot PNG with per-CpG significance marks and '_psign' suffix (e.g. Test sequence_meth_profile_collection1_proportional_psign.png)
- Per-replicate lollipop plots PNG (e.g. Test sequence_lollipop_replicates_as-is_proportional.png)
- Per-CpG and mean-methylation boxplots
- Two-by-two Student's t-test comparative statistics (e.g. P = 2.34e−05)
- Grouped .html report written to the 'reports' folder; all outputs in the 'results' folder

## How to apply

After completing individual sample analyses in ABSP (which produce *_meth_table outputs such as 'CDH1_High methylated DNA_3_meth_table.png'), navigate to the Shiny app's 'Grouped analysis' tab and assign samples to collections, choosing plotting parameters such as CpG position labels (CpG numbers/None/arbitrary coordinates) and proportional or as-is scaling. Run the grouped analysis, which gathers samples, computes two-by-two Student's t-tests comparing CpG methylation between groups, and writes PNG plots to the 'results' folder and an .html report to the 'reports' folder. Key outputs are per-replicate lollipop plots (e.g. 'Test sequence_lollipop_replicates_as-is_proportional.png') and the methylation profile plot with significance annotation for every CpG covered by the sequencing results (e.g. 'Test sequence_meth_profile_collection1_proportional_psign.png'). In the example comparison, low-methylated DNA averaged 1.43% (±1.87%) CpG methylation versus 95.54% (±0.67%) for high-methylated DNA, with a Student's t-test P-value of 2.34e−05 confirming the group difference.

## Related tools

- **ABSP** (R/Shiny tool whose 'Grouped analysis' tab gathers samples, runs comparative statistics, and generates lollipop and significance-annotated methylation profile plots) — https://github.com/ABSP-methylation-tool/ABSP
- **Shiny** (Built-in R Shiny app providing the user-friendly 'Grouped analysis' tab interface with plotting parameter selection) — https://github.com/ABSP-methylation-tool/ABSP
- **RStudio** (Used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the ABSP app via the 'Run App' button)
- **R** (Underlying R environment in which ABSP's grouped analysis, t-tests, and plot generation execute)

## Evaluation signals

- The grouped-results directory (e.g. grouped_results_direct_20230609-1727) contains the lollipop plot PNG, the methylation profile PNG with the '_psign' significance suffix, and the grouped .html report
- Significance marks are present for every CpG site covered by the sequencing results in the methylation profile plot
- The grouped .html report in the 'reports' folder reproduces the comparative statistics, e.g. mean CpG methylation 1.43% (±1.87%) vs 95.54% (±0.67%) with t-test P = 2.34e−05 in the example
- Plot filenames reflect the selected scaling parameters (as-is/proportional) and collection number, matching the 'Test sequence' example naming pattern

## Limitations

- Grouped analysis requires all samples to have been individually analyzed beforehand; it consumes individual methylation tables, not raw traces
- If an arbitrary reference genome is used, coordinates in plots/tables are arbitrary and CpG position labels should be set to 'CpG numbers' or 'None' in the Grouped Analysis
- Trimming that leaves sequences too short can abort analyses ('no CpG sites were found covered by sequencing results'), leaving few CpG sites for grouped comparison

## Evidence

- [methods] Grouped analysis computes t-tests and lollipop plots: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [readme] README grouped-analysis outputs: "Navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [readme] Significance-annotated profile plot example: "Test sequence_meth_profile_collection1_proportional_psign.png"
- [abstract] ABSP input and methylation percentage comparison scope: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
