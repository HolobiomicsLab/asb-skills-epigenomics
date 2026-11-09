---
name: methylation-group-comparison-statistics
description: Use when ab1 bisulfite sequencing PCR reads have already been individually
  analyzed for CpG methylation and you need to compare methylation percentages between
  sample groups (e.g. high- vs low-methylated DNA controls) with plots and significance
  tests.
when_to_use_negative:
- Samples have not yet been individually analyzed by ABSP — run the 'Individual analysis'
  tab first, since grouped analysis gathers already-computed per-CpG methylation levels
- You need genome-wide methylation quantification rather than region-specific CpG
  methylation from BSP experiments — ABSP is designed for region-specific analysis
  from .ab1 trace files
- You only have sequencing data without raw trace files (e.g. already-called FASTQ
  alignments) — ABSP's workflow starts from .ab1 chromatogram traces
edam_operation: http://edamontology.org/operation_0324
tools:
- name: ABSP
  role: R tool providing the 'Grouped analysis' (and 'Multiple analyses') Shiny tabs
    that gather individually analyzed samples, generate lollipop/profile plots, and
    run comparative statistics between groups
  repo: https://github.com/ABSP-methylation-tool/ABSP
- name: R
  role: Language in which ABSP is implemented; grouping, plotting and statistical
    comparison scripts run in R
  repo: https://www.r-project.org/
- name: RStudio
  role: IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the ABSP
    Shiny app
  repo: https://www.rstudio.com/products/rstudio/download/
- name: shiny
  role: User-friendly interface of ABSP; the 'Grouped analysis' tab runs within the
    built-in Shiny app
- name: Gviz
  role: R package used by ABSP for genomic plots of the reference sequence alongside
    methylation results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad008
    title: 'ABSP: an automated R tool to efficiently analyze region-specific CpG methylation
      from bisulfite sequencing PCR'
  source_content_hash: sha256:4fd0c417b8f09e4a697218a207b963df48bf42df044083139955f1913948e85d
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
  edam_operation: http://edamontology.org/operation_0324
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# methylation-group-comparison-statistics

## Summary

Grouped methylation comparison as implemented in ABSP: after individual direct-BSP/cloning-BSP sample analysis, gather samples into groups, generate lollipop and methylation profile plots, and run comparative statistics on CpG methylation percentages between groups.

## When to use

Use when .ab1 bisulfite sequencing PCR reads have already been individually analyzed for CpG methylation and you need to compare methylation percentages between sample groups (e.g. high- vs low-methylated DNA controls) with plots and significance tests.

## When NOT to use

- Samples have not yet been individually analyzed by ABSP — run the 'Individual analysis' tab first, since grouped analysis gathers already-computed per-CpG methylation levels
- You need genome-wide methylation quantification rather than region-specific CpG methylation from BSP experiments — ABSP is designed for region-specific analysis from .ab1 trace files
- You only have sequencing data without raw trace files (e.g. already-called FASTQ alignments) — ABSP's workflow starts from .ab1 chromatogram traces

## Inputs

- Individually analyzed ABSP sample results (per-CpG methylation tables) stored in the 'results' folder
- Grouping definitions for samples (e.g. high-methylated vs low-methylated CDH1 promoter DNAs, EpigenDx 80-8061-HGHM5 / 80-8062-HGUM5)
- Optionally an input table of entries for the 'Multiple analyses' tab

## Outputs

- Lollipop plots per sample/replicate (e.g. *_lollipop_replicates_as-is_proportional.png)
- Methylation profile plots per collection (e.g. *_meth_profile_collection1_proportional_psign.png)
- Comparative statistics between groups (methylation percentage contrasts with P-values)
- .html analysis report in the 'reports' folder and output files in the 'results' folder

## How to apply

First complete the ABSP 'Individual analysis' tab per sample: load the reference sequence in .fasta format and the .ab1 trace files, trim reads by combining the Phred quality-score method (default th_quality_error = 0.001, i.e. 99.9% base-calling accuracy / Phred 30, line 607 of ABSP_individual_analysis.Rmd) with the primary peak-ratio method (th_mixed_position = 0.75, th_mixed_perc = 75%, lines 612/614), then compute per-CpG methylation levels. Then, in the 'Grouped analysis' tab, gather individually analyzed samples (e.g. high-methylated vs unmethylated CDH1 promoter control DNAs), choose plotting parameters (CpG position labels, proportional vs absolute), and run the analysis to generate lollipop plots and methylation profile plots plus comparative statistics between groups. Review outputs in the generated .html report and the 'results' folder. Validate by reproducing known contrasts: for the EpigenDx CDH1 example, CpG #4 methylation 0.47% ± 0.81% vs 81.44% ± 3.96% (P = 0.00051) and mean methylation 1.43% ± 1.87% vs 95.54% ± 0.67% (P = 2.34e-05). For multiple samples or multiple grouping schemes, use the 'Multiple analyses' tab with filled input tables to launch everything in one click.

## Related tools

- **ABSP** (R tool providing the 'Grouped analysis' (and 'Multiple analyses') Shiny tabs that gather individually analyzed samples, generate lollipop/profile plots, and run comparative statistics between groups) — https://github.com/ABSP-methylation-tool/ABSP
- **R** (Language in which ABSP is implemented; grouping, plotting and statistical comparison scripts run in R) — https://www.r-project.org/
- **RStudio** (IDE used to open 'ABSP Rproject.Rproject' and 'app.R' and launch the ABSP Shiny app) — https://www.rstudio.com/products/rstudio/download/
- **shiny** (User-friendly interface of ABSP; the 'Grouped analysis' tab runs within the built-in Shiny app)
- **Gviz** (R package used by ABSP for genomic plots of the reference sequence alongside methylation results)

## Examples

```
shiny::runApp("app.R")  # in RStudio from the ABSP Rproject; then use the 'Grouped analysis' tab to gather samples and run comparative statistics
```

## Evaluation signals

- Expected numeric contrast reproduced: CpG site #4 methylation 0.47% ± 0.81% (low-methylated) vs 81.44% ± 3.96% (high-methylated) with P = 0.00051
- Mean methylation contrast reproduced: 1.43% ± 1.87% vs 95.54% ± 0.67% with P = 2.34e-05
- Lollipop plots and methylation profile plots are generated for each group/collection in the 'results' folder (e.g. *_lollipop_replicates_as-is_proportional.png)
- An .html report is written to the 'reports' folder after the grouped analysis completes
- Trimming/QC outcome is consistent with expectations (e.g. 5/6 forward reads fail trimming/QC in the CDH1 example) before grouping

## Limitations

- QC failures can reduce group membership: in the example direct-BSP CDH1 dataset, 5/6 forward reads fail trimming/QC, so only surviving reads contribute to group statistics
- Trimming thresholds may need adjustment: if quality-score trimming is too aggressive, lower th_quality_error (default 0.001); if primary-ratio trimming is too strict, adjust th_mixed_position (default 0.75) or th_mixed_perc (default 75%)
- A grouped analysis can be aborted with 'Error: Analysis has been stopped as no CpG sites were found covered by sequencing results' if trimmed sequences are too short
- If an arbitrary reference genome is used, arbitrarily set coordinates must not be taken into consideration in tables or plots; use 'CpG numbers' or 'None' for CpG position labels

## Evidence

- [methods] Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics: "Proceed to run the analysis which will gather samples, generate visualization plots (lollipop plots) and compare methylation data between groups using comparative statistics"
- [abstract] It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages.: "It uses the raw sequencing trace files (.ab1) as input to compute and compare CpG methylation percentages."
- [readme] After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting.: "After individually analyzing all of your samples, navigate to the "Grouped analysis" tab and complete input entries and select your parameters for plotting."
- [methods] The "Multiple analyses" tab is useful to analyze multiple samples and/or for multiple grouping analysis, launched in one click: "The "Multiple analyses" tab is useful to analyze multiple samples and/or for multiple grouping analysis, launched in one click"
- [readme] If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`: "If the trimming method based on Phred quality scores is too aggressive, consider lowering the minimum base-calling error probability threshold `th_quality_error`"
