---
name: orientation-barplot-visualization
description: Use when an Asymmetron strand-asymmetry analysis has produced per-feature
  same/opposite counts or ratios (e.g. a tfbs_chipseq_strand_asymmetries.tsv table)
  and the results need visual comparison across features.
when_to_use_negative:
- The data are raw unoriented BED intervals that have not yet been processed by Asymmetron's
  orientation/contained_asymmetries functions — no counts exist to plot
- The goal is genome-wide visualization of asymmetry along a coordinate axis (e.g.
  replication timing profiles) rather than categorical same/opposite count comparison
  per feature
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Produces the strand asymmetry counts/ratios and TSV tables that the barplots
    visualize
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: matplotlib
  role: Rendering library for the orientation count barplots
- name: seaborn
  role: High-level barplot API used on top of matplotlib for grouped same/opposite
    counts
- name: pandas
  role: Loads the tfbs_chipseq_strand_asymmetries.tsv table into a DataFrame for plotting
- name: scipy
  role: Supplies the two-tailed binomial test P-values (p = 0.5) annotated on the
    barplots
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:5e2470930400e4f9f20615ae8c0f8b0a4621139ad6220468819e6ec403d81d53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - matplotlib
  - numpy
  - pandas
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# orientation-barplot-visualization

## Summary

Generate barplots of same-versus-opposite strand orientation counts (or asymmetry ratios) for each TF/feature comparison using matplotlib and seaborn, accompanying the binomial-test asymmetry tables produced by Asymmetron analyses such as TFBS orientation within ChIP-seq peaks.

## When to use

Use when an Asymmetron strand-asymmetry analysis has produced per-feature same/opposite counts or ratios (e.g. a tfbs_chipseq_strand_asymmetries.tsv table) and the results need visual comparison across features.

## When NOT to use

- The data are raw unoriented BED intervals that have not yet been processed by Asymmetron's orientation/contained_asymmetries functions — no counts exist to plot
- The goal is genome-wide visualization of asymmetry along a coordinate axis (e.g. replication timing profiles) rather than categorical same/opposite count comparison per feature

## Inputs

- Asymmetry table TSV (e.g. tfbs_chipseq_strand_asymmetries.tsv) with same/opposite strand counts, ratios, and binomial P-values per TFBS set
- Oriented CTCF ChIP-seq peak BED file and Unibind-derived TFBS BED files used by Asymmetron's contained_asymmetries function

## Outputs

- Grouped barplots (PNG/PDF) of same versus opposite orientation counts per transcription factor comparison, with Bonferroni-corrected significance indicated

## How to apply

After running contained_asymmetries (regions = oriented CTCF ChIP-seq peaks, motifs = Unibind TFBS BED files) and computing the same/opposite ratio as same/(same+opposite) per TFBS set, load the written asymmetry table (e.g. tfbs_chipseq_strand_asymmetries.tsv) with pandas. Build grouped barplots of same versus opposite orientation counts for each transcription factor comparison using matplotlib/seaborn, optionally annotating bars with the two-tailed binomial test P-values (p = 0.5) and their Bonferroni-adjusted values so significance is visible alongside effect size. Keep the default expected asymmetry of 0.5 as the visual reference point (a ratio of 0.5 means random orientation). Write plots alongside the TSV so each barplot maps 1:1 to a row of the asymmetry table.

## Related tools

- **Asymmetron** (Produces the strand asymmetry counts/ratios and TSV tables that the barplots visualize) — https://github.com/Ahituv-lab/Asymmetron
- **matplotlib** (Rendering library for the orientation count barplots)
- **seaborn** (High-level barplot API used on top of matplotlib for grouped same/opposite counts)
- **pandas** (Loads the tfbs_chipseq_strand_asymmetries.tsv table into a DataFrame for plotting)
- **scipy** (Supplies the two-tailed binomial test P-values (p = 0.5) annotated on the barplots)

## Examples

```
import pandas as pd, seaborn as sns; df = pd.read_csv('tfbs_chipseq_strand_asymmetries.tsv', sep='\t'); sns.barplot(x='tf', y='same', data=df)
```

## Evaluation signals

- Every TFBS comparison in the TSV table has a corresponding bar in the plot (bar count equals row count of tfbs_chipseq_strand_asymmetries.tsv)
- Same + opposite counts per feature sum to the total overlapping motif-peak pairs reported by contained_asymmetries for that feature
- Same/(same+opposite) ratios recomputed from the plotted counts match the table values; the expected-asymmetry reference is 0.5 (random orientation)
- Features flagged significant in the table (Bonferroni-corrected binomial P-value threshold) are visually distinguishable (e.g. asterisks or color) in the plot
- Re-running the plotting code on the same TSV reproduces identical figures (deterministic rendering)

## Limitations

- Barplots show counts/ratios only; significance depends on the upstream binomial test with Bonferroni correction and the default expected asymmetry of 0.5, so plots alone do not establish bias
- Comparisons with very few overlapping motif-peak pairs yield unstable bars and low-power binomial tests; TF PWMs with no motif matches were filtered out upstream (leaving 551 PWMs)
- The conda environment in the README pins python 3.5 with pybedtools/seaborn/numpy; version drift may alter seaborn plotting APIs

## Evidence

- [methods] visualizations were performed in Python with ‘matplotlib’ and ‘seaborn’ packages: "visualizations were performed in Python with ‘matplotlib’ and ‘seaborn’ packages"
- [methods] We then calculate the P-value for the two-tailed binomial test as follows: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’: "Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’"
- [readme] Asymmetron can identify strand asymmetries within consecutive occurrences of a single genomic element and for pairs of overlapping and non-overlapping genomic elements: [evidence span withheld — anchor status: external_doc]
