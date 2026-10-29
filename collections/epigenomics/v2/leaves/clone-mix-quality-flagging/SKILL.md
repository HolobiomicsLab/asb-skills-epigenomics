---
name: clone-mix-quality-flagging
description: Decision support for cloning-BSP grouped analysis when per-clone CpG
  methylation percentages need conversion into methylation statuses and mixed clones
  must be flagged before producing methylation_status_table.csv.
when_to_use_negative:
- Direct-BSP data where each trace file represents a single PCR product rather than
  a cloned amplicon — the mixed-clone concept does not apply
- Analyses where every CpG site in a clone falls cleanly into the 0-20% or 80-100%
  bins; no flagging decision is needed and the plain status assignment suffices
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3673
tools:
- name: ABSP
  role: R-based tool whose grouped analysis implements the per-clone methylation-status
    thresholds and >20% partial-CpG clone-mix flagging; provides the 'Grouped analysis'
    tab and the .Rmd workflow that converts individual methylation tables into methylation_status_table.csv
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Underlying language used to execute the ABSP grouped-analysis scripts (ABSP_grouped_analysis.Rmd)
    that perform the threshold conversion and flagging
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: RStudio
  role: IDE used to open the ABSP Rproject and run the Shiny app that hosts the Grouped
    analysis tab performing the flagging
  repo: https://www.rstudio.com/products/rstudio/download/
- name: Shiny
  role: Built-in Shiny app provides the user-friendly 'Grouped analysis' interface
    where the per-clone status thresholds are applied
  repo: https://github.com/ABSP-methylation-tool/ABSP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:a76ba7d2c230f1677bb98620fac239314295b12850f92b1a3eb85ded2fe84e06
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ABSP-methylation-tool/ABSP
  tools:
  - ABSP
  - R
  - RStudio
  - Shiny
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clone-mix-quality-flagging

## Summary

Flags potentially mixed clones in cloning-BSP grouped analysis by applying ABSP's fixed methylation-status thresholds: 0-20% unmethylated, 80-100% methylated, 20-80% partial methylation annotated as NA, and >20% partially methylated CpGs triggering a clone-mix flag.

## When to use

Decision support for cloning-BSP grouped analysis when per-clone CpG methylation percentages need conversion into methylation statuses and mixed clones must be flagged before producing methylation_status_table.csv.

## When NOT to use

- Direct-BSP data where each trace file represents a single PCR product rather than a cloned amplicon — the mixed-clone concept does not apply
- Analyses where every CpG site in a clone falls cleanly into the 0-20% or 80-100% bins; no flagging decision is needed and the plain status assignment suffices

## Inputs

- Per-sample individual methylation tables produced by the ABSP 'Individual analysis' tab (clone x CpG methylation percentage tables)
- Sample grouping definition used in the 'Grouped analysis' tab

## Outputs

- methylation_status_table.csv with per-clone status annotations (unmethylated / methylated / NA) across samples within each group
- Clone-mix flag annotations for clones exceeding 20% partially covered CpGs

## How to apply

In the ABSP grouped samples analysis, load the per-sample individual methylation tables produced for each sample group from the ABSP 'examples' folder of deposited example data. Convert each clone's CpG methylation percentage into a status using the fixed thresholds: 0-20% unmethylated, 80-100% methylated, and 20-80% (partial methylation) annotated as not available. Flag clones with more than 20% partially covered CpGs as potential mixes of clones, and annotate all data for these flagged clones as not available. Aggregate the per-clone status annotations across samples within each group and export the resulting table as methylation_status_table.csv. This flagging step is necessary because ABSP's cloning-BSP workflow cannot distinguish a truly partially methylated clone from a mixed sample, so flagging protects downstream methylation-status interpretation.

## Related tools

- **ABSP** (R-based tool whose grouped analysis implements the per-clone methylation-status thresholds and >20% partial-CpG clone-mix flagging; provides the 'Grouped analysis' tab and the .Rmd workflow that converts individual methylation tables into methylation_status_table.csv) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Underlying language used to execute the ABSP grouped-analysis scripts (ABSP_grouped_analysis.Rmd) that perform the threshold conversion and flagging) — https://github.com/ABSP-methylation-tool/ABSP
- **RStudio** (IDE used to open the ABSP Rproject and run the Shiny app that hosts the Grouped analysis tab performing the flagging) — https://www.rstudio.com/products/rstudio/download/
- **Shiny** (Built-in Shiny app provides the user-friendly 'Grouped analysis' interface where the per-clone status thresholds are applied) — https://github.com/ABSP-methylation-tool/ABSP

## Examples

```
After running the ABSP 'Grouped analysis' tab (in RStudio: open app.R → Run App → Grouped analysis tab → Run analysis), the tool writes methylation_status_table.csv into your "results" folder.
```

## Evaluation signals

- methylation_status_table.csv exists and its status field contains only unmethylated, methylated, or NA values
- Every clone row in the output table carries a status annotation — no missing values
- Clones flagged as potential clone mixes carry the >20% partial-CpG annotation and their data is set to not available
- Status assignments are consistent with the ABSP thresholds (0-20% unmethylated, 80-100% methylated, 20-80% NA) when cross-checked against the ABSP 'examples' folder grouped-analysis outputs
- Aggregation across samples within each group yields one row per clone with group-consistent flagging

## Limitations

- Partial methylation (20-80%) is deliberately annotated as not available, so mixed-clone detection relies on the 20% partial-CpG count heuristic and may misclassify clones whose true methylation is intermediate but uniform
- The flag is a heuristic — it cannot confirm a mixed sample, only flag potential clone mixes for manual inspection
- Coordinates and CpG labels in grouped plots and tables are affected by reference-genome availability; the same caveat applies to the aggregated status table when using an arbitrary genome

## Evidence

- [other] ABSP thresholds for per-clone status conversion: [evidence span withheld — anchor status: not_found]
- [other] Clone-mix flag rule: [evidence span withheld — anchor status: not_found]
- [readme] Grouped analysis tab description: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [readme] Analysis prerequisites — individual tables exist: "Proceed to run the analysis to compute the CpG methylation levels of your sample"
- [readme] Output artifacts of grouped analysis: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [readme] Result location for verifying the output: "Once the analysis is complete, review the results in the .html report generated in your "reports" folder"
