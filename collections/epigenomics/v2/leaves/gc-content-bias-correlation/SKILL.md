---
name: gc-content-bias-correlation
description: Use when comparing bias correction across organisms or protocols to decide
  whether GC normalization suffices as a bias proxy, or when diagnosing whether observed
  expression shifts in RNA-Seq track transcript GC content.
when_to_use_negative:
- The analysis goal is improving expression accuracy via qRT-PCR benchmarking rather
  than characterizing whether GC content tracks sequence-specific bias — use the qRT-PCR
  correlation skill instead
- Only a single organism/protocol is available and no cross-organism contrast (e.g.
  human vs. yeast) is intended, so the proxy-vs-direct comparison cannot be framed
- The data are not RNA-Seq fragment alignments (e.g. NanoString counts or microarray
  intensities), for which effective-length bias is undefined
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3574
tools:
- name: Python
  role: Converts standard SAM and FASTA input into the mseq input format and computes
    per-transcript log fold change in effective length and per-dataset correlations
    (pandas/numpy)
- name: eXpress-D
  role: Produces the bias-corrected fragment assignment / effective-length estimates
    whose delta against uncorrected runs defines the sequence-specific bias signal
  repo: https://github.com/adarob/express-d
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:e3539001c561c19f9627131970330eae57f314077b96a1c3d696896c89f1314e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adarob/express-d
  tools:
  - Python
  - eXpress-D
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3574
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gc-content-bias-correlation

## Summary

Quantifies whether GC-content-derived bias acts as a proxy for sequence-specific fragment bias in RNA-Seq by computing the correlation between the log fold change in effective length (bias-corrected vs. uncorrected eXpress/eXpress-D estimates) and transcript GC content, per dataset.

## When to use

Use when comparing bias correction across organisms or protocols to decide whether GC normalization suffices as a bias proxy, or when diagnosing whether observed expression shifts in RNA-Seq track transcript GC content.

## When NOT to use

- The analysis goal is improving expression accuracy via qRT-PCR benchmarking rather than characterizing whether GC content tracks sequence-specific bias — use the qRT-PCR correlation skill instead
- Only a single organism/protocol is available and no cross-organism contrast (e.g. human vs. yeast) is intended, so the proxy-vs-direct comparison cannot be framed
- The data are not RNA-Seq fragment alignments (e.g. NanoString counts or microarray intensities), for which effective-length bias is undefined

## Inputs

- RNA-Seq alignment/fragment data (e.g. SRA012427 human, SRA020818_RH yeast)
- Bias-corrected eXpress/eXpress-D (Cufflinks) output files with per-transcript effective lengths
- Uncorrected expression estimates with per-transcript effective lengths
- Transcript sequences or reference FASTA for computing GC content
- mseq-format input generated from standard SAM and FASTA files (via the provided Python conversion script)

## Outputs

- Per-dataset table of GC-content vs. transcript-bias correlation coefficients (bias_correlations.csv)
- Per-transcript log fold change in effective length (bias-corrected vs. uncorrected)
- Qualitative verdict per dataset: GC correction proxies sequence-specific bias (strong positive correlation) or does not (weak correlation)

## How to apply

Retrieve the RNA-Seq datasets of interest (e.g. human SRA012427 and yeast SRA020818_RH) together with their bias-corrected and uncorrected Cufflinks/eXpress output files. For each transcript, compute bias as the log fold change in effective length between the bias-corrected and uncorrected estimates using Python (pandas/numpy), which leverages eXpress's likelihood-based framework for fragment-bias estimation. Compute the correlation between the GC-derived bias signal and the sequence-specific (effective-length) bias signal separately for each dataset, e.g. panel a (human) and panel b (yeast). Assemble the per-dataset correlation coefficients into a table (bias_correlations.csv) to show whether GC correction proxies sequence-specific bias. Judge success by validation criteria: the file exists with one correlation row per dataset, and the human correlation is strong/positive while the yeast correlation is weak — the paper's reported contrast between organisms.

## Related tools

- **Python** (Converts standard SAM and FASTA input into the mseq input format and computes per-transcript log fold change in effective length and per-dataset correlations (pandas/numpy))
- **eXpress-D** (Produces the bias-corrected fragment assignment / effective-length estimates whose delta against uncorrected runs defines the sequence-specific bias signal) — https://github.com/adarob/express-d

## Examples

```
python compute_bias_gc_correlation.py --corrected cufflinks_biascorrected.csv --uncorrected cufflinks_uncorrected.csv --fasta transcripts.fa --organism human --out bias_correlations.csv
```

## Evaluation signals

- bias_correlations.csv exists and contains one row per dataset with a finite correlation coefficient
- The human (SRA012427) correlation between GC content and effective-length log fold change is positive and strong
- The yeast (SRA020818_RH) correlation is weak, reproducing the paper's reported contrast between the two organisms
- Per-transcript bias values equal log(effective_length_corrected / effective_length_uncorrected), so sign flips track direction of correction
- Correlations are computed per dataset (panel a and panel b separately), not pooled across organisms

## Limitations

- GC content is not always an effective marker of sequence-specific bias — it proxies fragment bias in human RNA-Seq (SRA012427) but fails in yeast (SRA020818_RH)
- The likelihood framework assumes bias must be estimated from fragment alignments within the same experiment, so correlations depend on the library preparation and sequencing technology used
- Bias estimation and expression estimation are linked; errors in either the corrected or uncorrected estimates propagate into the log fold change signal

## Evidence

- [results] Transcript bias correlates strongly with GC content in human RNA-Seq (SRA012427), suggesting GC corrections proxy sequence-specific bias correction: [evidence span withheld — anchor status: not_found]
- [supplementary] Python script that converts standard SAM and FASTA input into the mseq input format: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [full_text] compute bias as the log fold change in effective length between the bias-corrected and uncorrected estimates: [evidence span withheld — anchor status: not_found]
- [intro] the problems of bias estimation and expression estimation are fundamentally linked, and must be solved together: "Therefore the problems of bias estimation and expression estimation are fundamentally linked, and must be solved together."
- [readme] For current documentation, see the [eXpress-D Wiki]: "Distributed ambiguous fragment assignment using [eXpress](http://bio.math.berkeley.edu/eXpress) and [Spark](http://spark.incubator.apache.org/)."
