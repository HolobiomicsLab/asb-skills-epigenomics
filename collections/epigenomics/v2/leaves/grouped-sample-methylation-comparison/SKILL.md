---
name: grouped-sample-methylation-comparison
description: Use when individually analyzed BSP samples (from .ab1 traces) must be
  gathered into groups to compare CpG methylation and assign per-clone statuses (0–20%
  unmethylated, 80–100% methylated) in ABSP.
when_to_use_negative:
- Samples have not yet gone through ABSP individual analysis — grouped analysis consumes
  per-sample methylation tables, not raw .ab1 traces
- Only a single sample needs CpG methylation-level computation with no group comparison
  — the 'Individual analysis' tab covers that
- Methylation data come from non-BSP platforms (e.g. arrays or whole-genome bisulfite
  pipelines); ABSP is scoped to region-specific CpG methylation from bisulfite sequencing
  PCR
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: ABSP
  role: Provides the 'Grouped analysis' tab that gathers individually analyzed samples,
    assigns per-clone methylation statuses via fixed thresholds, and compares groups
    (active version v1.2.2)
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment executing the ABSP scripts (app.R, ABSP_grouped_analysis.Rmd)
  repo: https://www.r-project.org/
- name: Shiny
  role: Built-in graphical interface hosting ABSP's 'Grouped analysis' tab
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:0ef70401fb8a62a19d542b6274922304c9bfe407046058185e39f6e19d8fd2bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# grouped-sample-methylation-comparison

## Summary

ABSP grouped-analysis workflow: gather individually analyzed BSP samples, convert per-clone CpG methylation percentages into statuses (0–20% unmethylated, 80–100% methylated, 20–80% NA, >20% partial CpGs = potential clone mix), and compare groups via lollipop plots and comparative statistics.

## When to use

Use when individually analyzed BSP samples (from .ab1 traces) must be gathered into groups to compare CpG methylation and assign per-clone statuses (0–20% unmethylated, 80–100% methylated) in ABSP.

## When NOT to use

- Samples have not yet gone through ABSP individual analysis — grouped analysis consumes per-sample methylation tables, not raw .ab1 traces
- Only a single sample needs CpG methylation-level computation with no group comparison — the 'Individual analysis' tab covers that
- Methylation data come from non-BSP platforms (e.g. arrays or whole-genome bisulfite pipelines); ABSP is scoped to region-specific CpG methylation from bisulfite sequencing PCR

## Inputs

- Per-sample methylation tables produced by the ABSP individual analysis of .ab1 sequencing trace files
- Reference sequence in .fasta format and sequencing files in .ab1 format (upstream individual-analysis inputs)
- Group-assignment input entries and plotting parameters for the ABSP 'Grouped analysis' tab
- Example input/output data in the ABSP 'examples' folder for calibration and cross-checking

## Outputs

- methylation_status_table.csv with per-clone statuses restricted to unmethylated / methylated / NA plus potential clone-mix flags
- Lollipop plots (visualization plots) gathering samples per group
- Comparative statistics comparing methylation data between groups
- Grouped-analysis .html report in the 'reports' folder and output files in the 'results' folder

## How to apply

First complete the ABSP individual analysis for every sample (reference .fasta plus .ab1 trace files), because the 'Grouped analysis' tab only gathers already-computed per-sample methylation tables. In the Shiny app, open the 'Grouped analysis' tab, complete the input entries (sample grouping) and select your plotting parameters, then run the analysis. For cloning-BSP data, classify each clone's CpG methylation percentage with the fixed thresholds — 0–20% unmethylated, 80–100% methylated, 20–80% partial methylation annotated as not available — and flag clones with more than 20% partially methylated CpGs as potential mixes of clones, annotating all their data as NA to avoid over-interpreting contaminated clones. Aggregate the per-clone status annotations across samples within each group and export the table as methylation_status_table.csv, alongside lollipop plots and comparative statistics between groups. Validate by cross-checking statuses and clone-mix flags against the example grouped-analysis outputs shipped in the ABSP 'examples' folder.

## Related tools

- **ABSP** (Provides the 'Grouped analysis' tab that gathers individually analyzed samples, assigns per-clone methylation statuses via fixed thresholds, and compares groups (active version v1.2.2)) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment executing the ABSP scripts (app.R, ABSP_grouped_analysis.Rmd)) — https://www.r-project.org/
- **Shiny** (Built-in graphical interface hosting ABSP's 'Grouped analysis' tab)

## Examples

```
shiny::runApp("app.R")
```

## Evaluation signals

- methylation_status_table.csv exists and its status field is restricted to the values unmethylated / methylated / NA
- Every clone row in the status table carries a status annotation (no unannotated rows)
- Status assignments match the fixed thresholds: 0–20% unmethylated, 80–100% methylated, 20–80% annotated as not available
- Clones with more than 20% partially methylated CpGs carry the potential clone-mix flag with all their data annotated NA
- Grouped outputs (lollipop plots, comparative statistics) are consistent with the example grouped-analysis results in the ABSP 'examples' folder

## Limitations

- Partially methylated CpGs (20–80%) and clones flagged as potential clone mixes (>20% partial CpGs) are annotated 'not available', so partial-methylation and mixed-clone information is dropped from the status table
- Grouped analysis requires all samples to have completed individual analysis first; on Linux, the package installation during the first launch of ABSP may take a considerable amount of time
- With an arbitrary reference genome, coordinates in output tables/plots must not be taken into consideration; prefer the 'CpG numbers' or 'None' CpG position label options in Grouped Analysis

## Evidence

- [methods] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [readme] After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting.: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [readme] You can locate example data for both inputs and outputs in the "examples" folder.: "You can locate example data for both inputs and outputs in the "examples" folder."
- [other] methylation percentages are converted to statuses by fixed thresholds: 0–20% is unmethylated, 80–100% is methylated, 20–80% (partial methylation) is annotated as not available: [evidence span withheld — anchor status: not_found]
- [other] a clone with more than 20% partially methylated CpGs is flagged as a potential mix of clones with all its data annotated as not available: [evidence span withheld — anchor status: not_found]
