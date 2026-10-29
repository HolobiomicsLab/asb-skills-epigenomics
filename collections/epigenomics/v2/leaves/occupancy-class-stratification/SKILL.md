---
name: occupancy-class-stratification
description: Use when you have ChIP-seq peak/occupancy data for many transcription
  factors and need to partition regions into HOT, MILD and COLD percentile classes
  to test whether apparent high occupancy correlates with control or artifact signal.
when_to_use_negative:
- The analysis goal is calling individual TF peaks or motifs rather than comparing
  signal enrichment between occupancy classes
- Input is already a per-class feature/enrichment table with no underlying bigWig
  signal or region intervals to extract from
- The ChIP-seq data lack matched mock/IgG control experiments, so class-wise artifact
  signal cannot be assessed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Scripting language used to orchestrate the stratification and signal-extraction
    workflow
- name: pyBedTools
  role: Loading and manipulating the HOT/MILD/COLD region interval sets (BED files)
- name: pyBigWig
  role: Extracting normalized IgG-control signal over +/-1 kb windows around region
    midpoints from bigWig tracks
- name: pandas
  role: Aggregating per-binned-position signal across regions per class and computing
    mean IgG signal summaries; writes igg_signal_summary.csv
- name: matplotlib
  role: Plotting smoothed HOT/MILD/COLD enrichment profiles per biosample group
- name: HOT-region-main.R
  role: Repository R script used for the HOT regions analysis underlying the manuscript
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:730567d475357d38d12b2230bd0ec7c213c30cc1210130aca77d3a4f396ec269
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - HOT-region-main.R
  - Python
  - matplotlib
  - pandas
  - pyBedTools
  - pyBigWig
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# occupancy-class-stratification

## Summary

Stratifies genomic regions into occupancy classes — HOT (top >99th percentile of transcription factor binding events), MILD (75th–99th) and COLD (<75th) — so that per-class enrichment of ChIP-seq signal (e.g., IgG mock controls) can be computed and compared to test whether high occupancy reflects ChIP-seq artifacts.

## When to use

Use when you have ChIP-seq peak/occupancy data for many transcription factors and need to partition regions into HOT, MILD and COLD percentile classes to test whether apparent high occupancy correlates with control or artifact signal.

## When NOT to use

- The analysis goal is calling individual TF peaks or motifs rather than comparing signal enrichment between occupancy classes
- Input is already a per-class feature/enrichment table with no underlying bigWig signal or region intervals to extract from
- The ChIP-seq data lack matched mock/IgG control experiments, so class-wise artifact signal cannot be assessed

## Inputs

- TF binding/occupancy counts per genomic region (to compute 99th/75th percentile cutoffs)
- Region interval sets as BED files for HOT (top >99th percentile), MILD (75th-99th) and COLD (<75th) classes
- Normalized ChIP-seq signal bigWig tracks from ENCODE IgG-control experiments (e.g., antibody ENCAB000AOJ) with experiment metadata grouped by biosample term

## Outputs

- Per-binned-position mean enrichment profiles for HOT, MILD and COLD regions per biosample group
- Per-class signal summary table (e.g., igg_signal_summary.csv with mean IgG signal per occupancy class)
- Smoothed matplotlib enrichment-profile plots comparing HOT vs MILD vs COLD classes

## How to apply

Define region classes by occupancy percentile computed from the number of transcription factor binding events per region: HOT regions are those above the 99th percentile, MILD regions fall between the 75th and 99th percentiles, and COLD regions lie below the 75th; store each class as an interval set (e.g., BED files) and load them with pyBedTools or pandas. For each class, extract normalized ChIP-seq signal (e.g., ENCODE IgG-control bigWig tracks such as antibody ENCAB000AOJ) over a +/-1 kb window anchored on the interval midpoint using pyBigWig, aggregating signal per binned position across regions to build mean enrichment profiles. Compute a per-class summary statistic such as mean IgG signal with pandas, and plot smoothed HOT/MILD/COLD profiles per biosample group with matplotlib to see whether HOT regions are consistently, though weakly, enriched in mock-control signal. The rationale is the article's finding that HOT regions are likely ChIP-seq artifacts ('hyper-ChIPable' regions), so enrichment in IgG controls provides evidence that unspecific antibody binding contributes to their apparent high occupancy.

## Related tools

- **Python** (Scripting language used to orchestrate the stratification and signal-extraction workflow)
- **pyBedTools** (Loading and manipulating the HOT/MILD/COLD region interval sets (BED files))
- **pyBigWig** (Extracting normalized IgG-control signal over +/-1 kb windows around region midpoints from bigWig tracks)
- **pandas** (Aggregating per-binned-position signal across regions per class and computing mean IgG signal summaries; writes igg_signal_summary.csv)
- **matplotlib** (Plotting smoothed HOT/MILD/COLD enrichment profiles per biosample group)
- **HOT-region-main.R** (Repository R script used for the HOT regions analysis underlying the manuscript) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
bw = pyBigWig.open('IgG_ENCAB000AOJ.bw'); sig = bw.values(chrom, mid-1000, mid+1000)
```

## Evaluation signals

- Percentile cutoffs reproduce the intended class sizes: HOT regions are strictly those above the 99th percentile of occupancy, MILD between the 75th and 99th, COLD below the 75th
- Extracted signal profiles cover a symmetric +/-1 kb window centered on each interval midpoint, with per-position signal aggregated across all regions in each class
- The HOT-class mean IgG signal exceeds MILD and COLD class means (consistent but weak enrichment), and the magnitude of this ordering is checked per biosample group
- igg_signal_summary.csv contains a mean IgG signal value per occupancy class and biosample group with no missing class entries
- Smoothed profiles plot successfully for every biosample-pooled bigWig, showing class separation rather than overlapping curves

## Limitations

- IgG enrichment on HOT regions is weak and its magnitude varies depending on the cell type analyzed, so per-biosample variability can obscure the class signal
- With currently available data it is not possible to estimate the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes
- HOT regions lack canonical motifs for transcription factors thought to be bound there, so motif-based validation of class assignment is not possible

## Evidence

- [other] HOT regions showed a consistent enrichment across multiple ENCODE IgG control experiments, but the enrichment was weak and its magnitude varied depending on the cell type analyzed.: [evidence span withheld — anchor status: not_found]
- [abstract] HOT regions are likely ChIP-seq artifacts similar to hyper-ChIPable regions: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [abstract] HOT regions lack canonical motifs: "Despite these features, HOT regions are solely defined using ChIP-seq experiments and shown to lack canonical motifs for transcription factors that are thought to be bound there."
- [discussion] cannot estimate antibody-specific error proportions: "With the currently available data, it is not possible to estimate the proportion of an antibody specific error resulting from the enrichment due to the pull-down of non-target proteins"
- [readme] repository scripts used for HOT region analysis: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
