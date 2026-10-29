---
name: chipseq-control-signal-profiling
description: Use when you have region intervals stratified by TF occupancy (e.g. HOT
  >99th percentile, MILD 75th-99th, COLD <75th) plus IgG-control bigWig tracks, and
  need to quantify/plot class-wise control enrichment.
when_to_use_negative:
- The goal is estimating what fraction of antibody-specific error comes from pull-down
  of non-target proteins vs. direct binding to polynucleotide epitopes — the paper
  states this is not possible with currently available data
- Input tracks are TF ChIP-seq samples rather than IgG/mock controls, so control ('hyper-ChIPable')
  enrichment cannot be assessed
- A KO ChIP-seq experiment shows no signal — lack of signal may reflect technical
  conditions (low read number, low library complexity, unsuccessful IP) rather than
  the artifact being probed here
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: pyBigWig
  role: Extracts normalized IgG signal over +/-1 kb windows anchored on region midpoints
    from ENCODE bigWig tracks
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
- name: pyBedTools
  role: Loads and manipulates HOT/MILD/COLD BED interval sets
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
- name: pandas
  role: Aggregates per-binned-position signal across regions per occupancy class and
    writes igg_signal_summary.csv
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
- name: matplotlib
  role: Plots smoothed HOT/MILD/COLD IgG enrichment profiles per biosample group
  repo: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:c9beca8905a24f81cac0355fb21b856a7ba2a40d78351e793f3a0d3cd6dba857
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
  tools:
  - Python
  - matplotlib
  - pandas
  - pyBedTools
  - pyBigWig
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# chipseq-control-signal-profiling

## Summary

Profiles IgG mock-control ChIP-seq signal over HOT, MILD and COLD occupancy-region classes from ENCODE controls to test whether unspecific antibody binding ('hyper-ChIPable' behavior) contributes to the apparent high occupancy of HOT regions.

## When to use

Use when you have region intervals stratified by TF occupancy (e.g. HOT >99th percentile, MILD 75th-99th, COLD <75th) plus IgG-control bigWig tracks, and need to quantify/plot class-wise control enrichment.

## When NOT to use

- The goal is estimating what fraction of antibody-specific error comes from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes — the paper states this is not possible with currently available data
- Input tracks are TF ChIP-seq samples rather than IgG/mock controls, so control ('hyper-ChIPable') enrichment cannot be assessed
- A KO ChIP-seq experiment shows no signal — lack of signal may reflect technical conditions (low read number, low library complexity, unsuccessful IP) rather than the artifact being probed here

## Inputs

- Normalized ENCODE IgG-control ChIP-seq bigWig signal tracks (e.g. antibody ENCAB000AOJ) with per-biosample metadata
- BED files of HOT (>99th percentile), MILD (75th-99th) and COLD (<75th percentile) occupancy region interval sets
- ENCODE portal metadata listing ChIP-seq experiments by biosample term

## Outputs

- Mean IgG enrichment profiles per occupancy class (HOT/MILD/COLD) per biosample group, binned over +/-1 kb around region midpoints
- igg_signal_summary.csv with per-class mean IgG signal summaries
- matplotlib plots of smoothed HOT/MILD/COLD enrichment profiles per biosample group

## How to apply

Query the ENCODE portal for ChIP-seq experiments using the IgG-control antibody ENCAB000AOJ and download normalized bigWig signal tracks with metadata, grouping experiments by biosample term. Load the HOT (top >99th percentile), MILD (75th-99th) and COLD (<75th) interval sets as BED files with pyBedTools/pandas. For each biosample-pooled IgG bigWig, extract normalized signal over a +/-1 kb window anchored on every interval midpoint using pyBigWig, then aggregate per-binned-position signal across regions within each occupancy class to build mean enrichment profiles. Compute a class-wise summary statistic (mean IgG signal per class), plot smoothed HOT/MILD/COLD profiles per biosample group with matplotlib, and write igg_signal_summary.csv. Interpretation follows the paper's rationale: a consistent but weak, cell-type-variable IgG enrichment on HOT regions relative to MILD/COLD bins supports the conclusion that HOT regions are likely ChIP-seq artifacts ('hyper-ChIPable' regions), though the proportion of antibody-specific error cannot be estimated from available data.

## Related tools

- **pyBigWig** (Extracts normalized IgG signal over +/-1 kb windows anchored on region midpoints from ENCODE bigWig tracks) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
- **pyBedTools** (Loads and manipulates HOT/MILD/COLD BED interval sets) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
- **pandas** (Aggregates per-binned-position signal across regions per occupancy class and writes igg_signal_summary.csv) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions
- **matplotlib** (Plots smoothed HOT/MILD/COLD IgG enrichment profiles per biosample group) — https://github.com/BIMSBbioinfo/HOT-or-not-examining-the-basis-of-high-occupancy-target-regions

## Examples

```
import pyBigWig; bw = pyBigWig.open('igg_control.bw'); sig = bw.values(chrom, mid-1000, mid+1000)
```

## Evaluation signals

- HOT regions show a mean IgG signal above both MILD and COLD classes in the enrichment profiles, confirming control-track enrichment
- The HOT-vs-MILD/COLD enrichment is weak in magnitude and varies across biosample groups rather than being uniform, matching the reported finding of 'consistent enrichment ... but the enrichment was weak and its magnitude varied depending on the cell type analyzed'
- igg_signal_summary.csv contains one mean-IgG-signal summary row per occupancy class per biosample group, with class labels restricted to HOT, MILD and COLD
- Signal windows are centered on interval midpoints and span +/-1 kb, so profiles should be symmetric around the region center
- All IgG bigWigs trace back to the specified control antibody (ENCAB000AOJ) in the ENCODE metadata, grouped by biosample term

## Limitations

- With currently available data it is not possible to estimate the proportion of an antibody specific error resulting from the enrichment due to the pull-down of non-target proteins vs. direct binding to polynucleotide epitopes
- Enrichment magnitude varies depending on the cell type analyzed, so conclusions from one biosample may not generalize
- IgG enrichment is weak, so noisy or low-depth control tracks may not resolve the HOT vs. MILD/COLD difference

## Evidence

- [other] HOT regions showed a consistent enrichment across multiple ENCODE IgG control experiments, but the enrichment was weak and its magnitude varied depending on the cell type analyzed.: [evidence span withheld — anchor status: not_found]
- [abstract] Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions.: "Here, we show that HOT regions are likely to be ChIP-seq artifacts and they are similar to previously proposed 'hyper-ChIPable' regions."
- [discussion] With the currently available data, it is not possible to estimate the proportion of an antibody specific error resulting from the enrichment due to the pull-down of non-target proteins vs. the direct: "With the currently available data, it is not possible to estimate the proportion of an antibody specific error resulting from the enrichment due to the pull-down of non-target proteins vs. the direct"
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
