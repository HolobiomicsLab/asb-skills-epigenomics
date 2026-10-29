---
name: clone-methylation-status-thresholding
description: Use when you have per-clone methylation percentage tables from ABSP individual
  cloning-BSP analyses and need to classify each clone into methylation statuses for
  a grouped analysis across samples.
when_to_use_negative:
- You are working with direct-BSP data rather than cloning-BSP clone data — direct-BSP
  grouped analysis compares methylation percentages between groups without per-clone
  status classification
- You need continuous methylation percentages for statistical comparison rather than
  discrete per-clone statuses (ABSP computes raw percentages in the individual analysis)
- Sequence has been trimmed too short by the Phred-quality (`th_quality_error`, default
  0.001) and primary-peak-ratio (`th_mixed_position`, default 0.75) trimming methods,
  leaving only a few covered CpG sites — rerun individual analysis first
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: ABSP
  role: Provides the grouped-analysis workflow that converts per-clone methylation
    percentages into statuses using fixed thresholds and flags potential clone mixes
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Language in which ABSP (ABSP_grouped_analysis.Rmd, ABSP_functions.R) is implemented
    and executed
  repo: https://www.r-project.org/
- name: Shiny
  role: User interface exposing the 'Grouped analysis' tab where the status conversion
    and aggregation are launched
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:dfdeb8a40e979d9cda76a0817a04dbe56c069614c270ac7609e99a78f77e0934
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# clone-methylation-status-thresholding

## Summary

Converts per-clone CpG methylation percentages from ABSP cloning-BSP individual analyses into discrete methylation statuses (unmethylated/methylated/NA) using fixed thresholds, and flags potential clone mixes, as implemented in the ABSP grouped analysis.

## When to use

Use when you have per-clone methylation percentage tables from ABSP individual cloning-BSP analyses and need to classify each clone into methylation statuses for a grouped analysis across samples.

## When NOT to use

- You are working with direct-BSP data rather than cloning-BSP clone data — direct-BSP grouped analysis compares methylation percentages between groups without per-clone status classification
- You need continuous methylation percentages for statistical comparison rather than discrete per-clone statuses (ABSP computes raw percentages in the individual analysis)
- Sequence has been trimmed too short by the Phred-quality (`th_quality_error`, default 0.001) and primary-peak-ratio (`th_mixed_position`, default 0.75) trimming methods, leaving only a few covered CpG sites — rerun individual analysis first

## Inputs

- Per-sample individual methylation tables (per-clone CpG methylation percentages) from ABSP individual analysis of .ab1 trace files
- ABSP 'examples' folder example data for cloning-BSP input/output
- Sample group definitions for the grouped analysis

## Outputs

- methylation_status_table.csv with a status field restricted to unmethylated/methylated/NA for every clone row
- Potential clone-mix flags (>20% partially methylated CpGs) on affected clone rows

## How to apply

Load the per-sample individual methylation tables produced for each sample group from the ABSP 'examples' folder of deposited example data, or from your own individual-analysis results. Convert each clone's CpG methylation percentage into a status using the ABSP fixed thresholds: 0–20% is unmethylated, 80–100% is methylated, and 20–80% (partial methylation) is annotated as not available (NA). Flag any clone with more than 20% partially methylated CpGs as a potential mix of clones and annotate all of its data as not available. Aggregate the per-clone status annotations across samples within each group (via the 'Grouped analysis' tab, which gathers samples and runs comparative statistics) and export the resulting table as methylation_status_table.csv.

## Related tools

- **ABSP** (Provides the grouped-analysis workflow that converts per-clone methylation percentages into statuses using fixed thresholds and flags potential clone mixes) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Language in which ABSP (ABSP_grouped_analysis.Rmd, ABSP_functions.R) is implemented and executed) — https://www.r-project.org/
- **Shiny** (User interface exposing the 'Grouped analysis' tab where the status conversion and aggregation are launched)

## Examples

```
Rscript -e "shiny::runApp('app.R')"  # then use the 'Grouped analysis' tab after completing individual analyses
```

## Evaluation signals

- The output table methylation_status_table.csv exists and its status field contains only the values unmethylated, methylated, or NA
- Every clone row in the output table is annotated with a status (no empty status cells)
- Clones with more than 20% partially methylated CpGs carry the potential clone-mix flag and have all data annotated as not available
- Status assignments match the fixed thresholds: 0–20% → unmethylated, 80–100% → methylated, 20–80% → NA
- Output is consistent with the grouped-analysis results shipped in the ABSP 'examples' folder

## Limitations

- The 20–80% range is deliberately annotated as not available rather than called 'partially methylated', so intermediate methylation levels carry no status
- Clones exceeding 20% partially methylated CpGs are treated as potential clone mixes and their data is entirely annotated as not available, discarding those measurements
- On Linux machines, package installation during the first ABSP launch may take a considerable amount of time

## Evidence

- [full_text] Grouped analysis status conversion thresholds: [evidence span withheld — anchor status: not_found]
- [readme] Grouped analysis gathers samples and compares data: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [readme] Grouped analysis follows individual analysis of all samples: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [abstract] ABSP input file format and purpose: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [readme] Example data availability for validation: "You can locate example data for both inputs and outputs in the "examples" folder."
