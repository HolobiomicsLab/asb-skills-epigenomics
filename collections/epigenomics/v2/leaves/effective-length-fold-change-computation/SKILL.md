---
name: effective-length-fold-change-computation
description: Use when you have paired bias-corrected and uncorrected Cufflinks/eXpress
  output per transcript and need to quantify how much fragment bias correction shifted
  effective lengths, or to test if GC content predicts the bias signal.
when_to_use_negative:
- Input is a single expression matrix with no paired corrected/uncorrected effective-length
  estimates, since the log fold change cannot be computed
- Analysis goal is direct expression-level accuracy assessment against qRT-PCR rather
  than characterizing the nature of the bias signal itself
- The library was already processed with a bias-correction method so both sides of
  the fold change are corrected and the quantity is trivially zero
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: eXpress-D
  role: Produces the bias-corrected expression estimates and effective lengths whose
    fold change against uncorrected values defines transcript bias
  repo: https://github.com/adarob/express-d/wiki
- name: Python
  role: Computes the per-transcript log fold change in effective length and per-dataset
    GC correlations (pandas/numpy); also converts SAM/FASTA into mseq format for eXpress
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:799401047477e15619d3e8eea831cf1e342bc23cbd6b5afb651a685edf84f8c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - eXpress-D
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# effective-length-fold-change-computation

## Summary

Quantifies per-transcript fragment bias in RNA-Seq as the log fold change in effective length between bias-corrected (eXpress/eXpress-D) and uncorrected expression estimates, then correlates this sequence-specific bias signal with transcript GC content to test whether GC normalization proxies bias correction (strong in human SRA012427, weak in yeast SRA020818_RH).

## When to use

Use when you have paired bias-corrected and uncorrected Cufflinks/eXpress output per transcript and need to quantify how much fragment bias correction shifted effective lengths, or to test if GC content predicts the bias signal.

## When NOT to use

- Input is a single expression matrix with no paired corrected/uncorrected effective-length estimates, since the log fold change cannot be computed
- Analysis goal is direct expression-level accuracy assessment against qRT-PCR rather than characterizing the nature of the bias signal itself
- The library was already processed with a bias-correction method so both sides of the fold change are corrected and the quantity is trivially zero

## Inputs

- Bias-corrected and uncorrected Cufflinks/eXpress transcript quantification files (effective lengths, FPKM) for the same library
- Per-transcript GC content table for the target transcriptome
- RNA-Seq datasets such as SRA012427 (human) and SRA020818_RH (yeast)

## Outputs

- Per-transcript bias values (log fold change in effective length) per dataset
- bias_correlations.csv — per-dataset correlation rows comparing sequence-specific and GC-derived bias signals

## How to apply

Retrieve the bias-corrected and uncorrected Cufflinks/eXpress output files for each RNA-Seq library (e.g. human SRA012427 and yeast SRA020818_RH). For each transcript, compute bias with Python (pandas/numpy) as the log fold change in effective length between the bias-corrected and uncorrected estimates, following the likelihood-based correction framework of the paper. Compute the Pearson correlation between this sequence-derived bias signal and a GC-derived bias marker separately per organism. Decision point: interpret a strong positive correlation as evidence that GC correction proxies sequence-specific bias correction, and a weak correlation as evidence GC is not an effective bias marker. Assemble per-dataset correlation coefficients into a table (bias_correlations.csv) contrasting the human and yeast results.

## Related tools

- **eXpress-D** (Produces the bias-corrected expression estimates and effective lengths whose fold change against uncorrected values defines transcript bias) — https://github.com/adarob/express-d/wiki
- **Python** (Computes the per-transcript log fold change in effective length and per-dataset GC correlations (pandas/numpy); also converts SAM/FASTA into mseq format for eXpress)

## Examples

```
df['bias'] = np.log2(df['eff_len_corrected'] / df['eff_len_uncorrected']); r = df['bias'].corr(df['gc_content'])
```

## Evaluation signals

- bias_correlations.csv exists and contains one correlation row per dataset (human panel a, yeast panel b)
- Human SRA012427 correlation between transcript bias and GC content is positive and strong, consistent with GC correction proxying sequence-specific bias
- Yeast SRA020818_RH correlation is weak, consistent with GC content not being an effective bias marker in that organism
- Per-transcript bias values equal the log fold change of effective length (corrected vs uncorrected) and are finite for all transcripts
- Uncorrected inputs reproduce uncorrected effective lengths, i.e. a transcript with no bias correction shows log fold change ≈ 0

## Limitations

- GC content is not always an effective bias marker — the human contrast fails in yeast (SRA020818_RH), so correlation strength is organism- and protocol-dependent
- The bias measure depends on the likelihood-based simultaneous estimation of bias and expression; fragment distribution non-uniformity must be accounted for during expression estimation or the effective lengths are meaningless
- Bias and expression estimation are fundamentally linked and must be solved together; partial or staggered estimates invalidate the fold-change interpretation

## Evidence

- [abstract] abstract: "This non-uniformity must be accounted for when estimating expression levels, and we show how to perform the needed corrections using a likelihood based approach"
- [intro] intro: "It is therefore desirable to estimate the extent and nature of bias indirectly by inferring it from the data (fragment alignments) in an experiment."
- [other] other: [evidence span withheld — anchor status: not_found]
- [readme] readme: "Distributed ambiguous fragment assignment using eXpress and Spark."
