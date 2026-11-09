---
name: cpg-methylation-table-processing
description: Use when you have per-clone CpG methylation percentages from ABSP individual
  analyses of cloning-BSP samples and need grouped-level status annotations for comparison
  across sample groups.
when_to_use_negative:
- Input is raw .ab1 trace files that have not yet been through ABSP individual analysis
  — run the individual analysis to compute methylation percentages first
- Data are direct-BSP (direct sequencing of PCR products) rather than cloning-BSP
  — the per-clone status/mix-flag conversion applies to clone-based data
- Goal is CpG-level methylation percentages rather than per-clone categorical statuses
  — use the individual analysis methylation tables directly
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: ABSP
  role: R tool whose grouped-analysis logic applies the 0–20/80–100/20–80 status thresholds
    and the >20% partial-CpG clone-mix flag to cloning-BSP data
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Runtime environment implementing the ABSP analysis scripts (ABSP_grouped_analysis.Rmd)
    that perform the status conversion and table export
  repo: https://www.r-project.org/
- name: Shiny
  role: Built-in ABSP Shiny app providing the 'Grouped analysis' tab where the grouped
    status analysis is launched
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:4454db17e5464d65afc88e728dc5f1a5bebe0e03463d2a28af22a9a2a029dfc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cpg-methylation-table-processing

## Summary

Converts per-clone CpG methylation percentages from cloning-BSP individual analyses into categorical methylation statuses (unmethylated/methylated/NA) using ABSP's fixed thresholds, flagging potential clone mixes, and exports a per-group methylation_status_table.csv.

## When to use

Use when you have per-clone CpG methylation percentages from ABSP individual analyses of cloning-BSP samples and need grouped-level status annotations for comparison across sample groups.

## When NOT to use

- Input is raw .ab1 trace files that have not yet been through ABSP individual analysis — run the individual analysis to compute methylation percentages first
- Data are direct-BSP (direct sequencing of PCR products) rather than cloning-BSP — the per-clone status/mix-flag conversion applies to clone-based data
- Goal is CpG-level methylation percentages rather than per-clone categorical statuses — use the individual analysis methylation tables directly

## Inputs

- Per-sample individual methylation tables from ABSP individual analysis of cloning-BSP samples (computed from .ab1 sequencing trace files and a reference .fasta)
- Sample grouping definitions (group entries for the ABSP 'Grouped analysis' tab)

## Outputs

- methylation_status_table.csv with per-clone statuses restricted to unmethylated/methylated/NA and clone-mix flags
- .html grouped-analysis report in the 'reports' folder

## How to apply

Load the per-sample individual methylation tables (derived from .ab1 trace files analyzed in the ABSP 'Individual analysis' tab) for each sample group. Convert each clone's CpG methylation percentage into a status using ABSP's fixed thresholds: 0–20% is unmethylated, 80–100% is methylated, and 20–80% (partial methylation) is annotated as not available. Flag any clone with more than 20% partially methylated CpGs as a potential mix of clones and annotate all its data as not available. Aggregate the per-clone status annotations across samples within each group (the 'Grouped analysis' tab gathers samples and compares methylation between groups) and export the resulting table as methylation_status_table.csv. Validate against the grouped-analysis outputs shipped in the ABSP 'examples' folder.

## Related tools

- **ABSP** (R tool whose grouped-analysis logic applies the 0–20/80–100/20–80 status thresholds and the >20% partial-CpG clone-mix flag to cloning-BSP data) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Runtime environment implementing the ABSP analysis scripts (ABSP_grouped_analysis.Rmd) that perform the status conversion and table export) — https://www.r-project.org/
- **Shiny** (Built-in ABSP Shiny app providing the 'Grouped analysis' tab where the grouped status analysis is launched) — https://github.com/ABSP-methylation-tool/ABSP

## Examples

```
Rscript -e "shiny::runApp('app.R')"  # open the ABSP Shiny app, then use the 'Grouped analysis' tab on the individually analyzed cloning-BSP samples
```

## Evaluation signals

- methylation_status_table.csv exists and its status field contains only unmethylated/methylated/NA values
- Every clone row in the output table is annotated with a status
- Clones with more than 20% partially methylated CpGs carry the potential-clone-mix flag and their data are annotated as not available
- 0–20% percentages map to unmethylated and 80–100% map to methylated with no boundary violations in the table
- Output is consistent with the example grouped-analysis outputs shipped in the ABSP 'examples' folder

## Limitations

- Partial methylation (20–80%) is annotated as not available, so intermediate methylation states are not represented in the status table
- Clones flagged as potential mixes of clones (>20% partially methylated CpGs) lose all their data to NA and cannot be interpreted
- Status conversion requires all samples to be individually analyzed first; grouped analysis gathers previously computed individual results

## Evidence

- [full_text] In the grouped analysis, methylation percentages are converted to statuses by fixed thresholds: 0–20% is unmethylated, 80–100% is methylated, 20–80% (partial methylation) is annotated as not available: [evidence span withheld — anchor status: not_found]
- [readme] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [methods] After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting.: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
