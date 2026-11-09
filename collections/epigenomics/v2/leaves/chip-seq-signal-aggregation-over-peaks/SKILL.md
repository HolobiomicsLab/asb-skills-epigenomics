---
name: chip-seq-signal-aggregation-over-peaks
description: 'Use when benchmarking peak callers on qbed calling-cards data and you
  need orthogonal validation: average ChIP-seq (bigWig) signal under each caller''s
  peaks plus pairwise peak-overlap concordance between callers.'
when_to_use_negative:
- Input is already a differential peak or gene-expression result — this skill validates
  peak-caller output quality, not downstream differential analysis
- No matching ChIP-seq bigWig signal track exists for the cell type and factor of
  the calling-cards data, since signal aggregation would be uninterpretable
- Peaks have not yet been called and filtered (pp.filter_peaks); aggregating signal
  over raw insertions rather than peak intervals defeats the comparison
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0769
tools:
- name: Pycallingcards
  role: 'Provides the pipeline: clean_qbed, call_peaks (CCcaller/MACCs), and plotting.calculate_signal
    for ChIP-seq signal under peaks'
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: bedtools
  role: Interval-overlap engine (with pybedtools) used to compute average ChIP-seq
    signal over called peak intervals
  repo: https://bedtools.readthedocs.io/en/latest/
- name: CCcaller
  role: Pycallingcards peak caller whose output peaks are benchmarked by ChIP-seq
    signal enrichment and overlap concordance
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: MACCs
  role: Second Pycallingcards peak caller benchmarked alongside CCcaller under default
    parameters
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Python
  role: Implementation language for the comparison workflow and CSV output
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:0c07924145129fd0b17c2c3f4db7e2a35c709648ef18776b85bcd8a9579c5565
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - MACCs
  - Pycallingcards
  - Python
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# chip-seq-signal-aggregation-over-peaks

## Summary

Aggregates ChIP-seq signal (e.g., H3K27ac, SP1 bigWig tracks) under called calling-cards peaks to quantify how well each peak caller's outputs are enriched for orthogonal TF-binding evidence, enabling quantitative comparison of CCcaller, MACCs, Blockify, and ccf_tools.

## When to use

Use when benchmarking peak callers on qbed calling-cards data and you need orthogonal validation: average ChIP-seq (bigWig) signal under each caller's peaks plus pairwise peak-overlap concordance between callers.

## When NOT to use

- Input is already a differential peak or gene-expression result — this skill validates peak-caller output quality, not downstream differential analysis
- No matching ChIP-seq bigWig signal track exists for the cell type and factor of the calling-cards data, since signal aggregation would be uninterpretable
- Peaks have not yet been called and filtered (pp.filter_peaks); aggregating signal over raw insertions rather than peak intervals defeats the comparison

## Inputs

- Peak sets (BED/peak dataframes) from CCcaller, MACCs, Blockify, Blockify_original, and ccf_tools
- qbed calling-cards insertions: Brd4 bulk CC HCT116 (GEO GSE156821) and SP1 scCC HCT116 (pycallingcards.datasets.SP1_K562HCT116_data)
- ChIP-seq bigWig signal tracks (H3K27ac, SP1) for HCT116
- bedtools/pybedtools installation for interval overlap

## Outputs

- peak_calling_comparison.csv with per-caller, per-dataset rows: average H3K27ac and SP1 ChIP-seq signal under peaks and pairwise peak-overlap percentages
- Caller ranking by ChIP-seq enrichment and concordance (analogous to Fig. 2c-e)

## How to apply

After calling peaks with pycallingcards.preprocessing.call_peaks (CCcaller or MACCs) and reference callers (Blockify, Blockify_original, ccf_tools) on cleaned qbed inputs (pycallingcards.preprocessing.clean_qbed), compute the average H3K27ac and SP1 ChIP-seq signal under each caller's peak intervals using pycallingcards.plotting.calculate_signal, which performs interval overlap against bigWig tracks via bedtools/pybedtools. Separately compute pairwise peak-overlap percentages between all caller pairs for both the no-background bulk dataset (Brd4 HCT116, GSE156821) and the with-background single-cell dataset (SP1 scCC HCT116 from datasets.SP1_K562HCT116_data). Write one row per caller per dataset containing average signal and overlap percentages to a CSV (e.g., peak_calling_comparison.csv). The rationale is that callers whose peaks concentrate stronger ChIP-seq signal and show higher mutual overlap are pinpointing TF binding sites more accurately.

## Related tools

- **Pycallingcards** (Provides the pipeline: clean_qbed, call_peaks (CCcaller/MACCs), and plotting.calculate_signal for ChIP-seq signal under peaks) — https://github.com/The-Mitra-Lab/pycallingcards
- **bedtools** (Interval-overlap engine (with pybedtools) used to compute average ChIP-seq signal over called peak intervals) — https://bedtools.readthedocs.io/en/latest/
- **CCcaller** (Pycallingcards peak caller whose output peaks are benchmarked by ChIP-seq signal enrichment and overlap concordance) — https://github.com/The-Mitra-Lab/pycallingcards
- **MACCs** (Second Pycallingcards peak caller benchmarked alongside CCcaller under default parameters) — https://github.com/The-Mitra-Lab/pycallingcards
- **Python** (Implementation language for the comparison workflow and CSV output)

## Examples

```
import pycallingcards as cc; qbed = cc.pp.clean_qbed(cc.datasets.SP1_K562HCT116_data()); peaks = cc.pp.call_peaks(qbed, method='CCcaller'); cc.pl.calculate_signal(peaks, bigwig='H3K27ac_HCT116.bw')
```

## Evaluation signals

- Output CSV exists with one row per peak caller per dataset (bulk no-background and single-cell with-background)
- Table contains average H3K27ac/SP1 signal fields and pairwise overlap-percentage fields for all caller pairs
- Overlap percentages reproduce the relative caller ranking shown in Fig. 2c-e (CCcaller and MACCs highly concordant; ccf_tools/Blockify more discordant)
- CCcaller and MACCs show higher ChIP-seq signal enrichment under peaks than the discordant callers
- clean_qbed was applied to insertions before peak calling, confirming inputs were preprocessed consistently across callers

## Limitations

- Comparison is run under default parameters for all five callers; different settings could change the concordance ranking
- Results are specific to the benchmarked HyPBase Brd4 bulk and SP1 single-cell HCT116 datasets and their matching ChIP-seq tracks; extrapolation to other factors/cell types is untested here
- bedtools (including pybedtools) must be installed as an external dependency for annotation and signal overlap

## Evidence

- [abstract] Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data.: "Pycallingcards introduces two innovative peak callers, CCcaller and MACCs, enhancing the accuracy and speed of pinpointing TF binding sites from CC data."
- [readme] Plotting module signal comparison: "show signal comparison with Chip-seq(BigWig file), display differential peaks, demonstrate potenial binding-gene expression relationships."
- [methods] bedtools dependency: "In order to complete the annotation, [bedtools](https://bedtools.readthedocs.io/en/latest/) (includes [pybedtools](https://daler.github.io/pybedtools/)) also needs to be installed."
- [methods] preprocessing functions: "**Preprocessing (pp)** helps to preprocess data from qbed/ccf data to call peaks, make annotation, make Anndata object and filter peaks."
