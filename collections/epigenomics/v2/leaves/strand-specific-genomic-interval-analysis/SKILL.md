---
name: strand-specific-genomic-interval-analysis
description: Use when you have per-window, per-nucleotide translocation-time curves
  from Tombo re-squiggled ONT reads labeled non-B (e.g. G-Quadruplex_Motif) vs B-DNA,
  and need strand-resolved divergence testing (e.g. reproducing Fig. 5).
when_to_use_negative:
- Input is raw nanopore current signal without per-base translocation times from Tombo
  re-squiggle — compute TT first as the ratio of current measurements to the ONT sampling
  rate
- The goal is non-B novelty detection/classification (GoFAE-DND, isolation forest,
  one-class SVM) rather than window-level divergence testing of TT curves
- Windows have mismatched lengths that cannot be aligned per non-B type, since IWT
  requires aligned curves within windows of matching length
edam_operation: http://edamontology.org/operation_2262
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: IWTomics
  role: Interval-wise testing permutation framework used to obtain pointwise and interval-wise
    adjusted p-values along the TT curves
- name: pandas
  role: Loads and structures the per-window per-nucleotide TT dataset with window
    labels, strand, and position
- name: numpy
  role: Numerical backing for the IWT permutation test on TT curves
- name: matplotlib
  role: Plots adjusted p-value trajectories and mean TT curve contrasts per type and
    strand
- name: seaborn
  role: Diagnostic plotting of IWT results
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:6433935b42900926ebbe9bdafdc5d5e222b464d51343af6ae296d0ab2aa7f3be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IWTomics
  - Python
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_2262
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# Strand-specific genomic interval analysis

## Summary

Statistically tests whether per-nucleotide ONT translocation-time (TT) curves measured in non-B DNA motif windows diverge from motif-free B-DNA control windows, separately per strand, using interval-wise testing (IWT) permutation p-values. This operationalizes the paper's finding that 'there exist significant differences between the timing of DNA translocation for non-B DNA bases compared with B-DNA'.

## When to use

Use when you have per-window, per-nucleotide translocation-time curves from Tombo re-squiggled ONT reads labeled non-B (e.g. G-Quadruplex_Motif) vs B-DNA, and need strand-resolved divergence testing (e.g. reproducing Fig. 5).

## When NOT to use

- Input is raw nanopore current signal without per-base translocation times from Tombo re-squiggle — compute TT first as the ratio of current measurements to the ONT sampling rate
- The goal is non-B novelty detection/classification (GoFAE-DND, isolation forest, one-class SVM) rather than window-level divergence testing of TT curves
- Windows have mismatched lengths that cannot be aligned per non-B type, since IWT requires aligned curves within windows of matching length

## Inputs

- Per-window per-nucleotide translocation-time (TT) dataset from task_001 (NA12878 / SRR15058166, Tombo re-squiggled FAST5), with window labels (non-B type vs B-DNA), strand, and position columns
- Window coordinates around non-B motifs extracted from the non-B DNA DB, extended to the opposite strand (windows of length 50 and 100)
- Motif-free B-DNA control windows (Step 5: 'Find motif free regions')

## Outputs

- iwt_results.csv — aggregated table of pointwise and interval-wise adjusted p-values reporting significant divergence per non-B type and per strand
- Diagnostic figures (adjusted p-value trajectories and mean TT curve contrasts per non-B type and strand) saved alongside iwt_results.csv in the results/ directory

## How to apply

Load the per-window per-nucleotide TT dataset produced by task_001 with pandas, retaining window labels (non-B type vs B-DNA), strand, and position. Align TT curves within windows of matching length for each non-B type (e.g. G-quadruplex motifs) and the motif-free B-DNA controls, computed separately for forward and reverse strands; positions are extended to the opposite strand per the window-preparation steps (Step 3 of the pipeline). Run the IWTomics interval-wise testing permutation test on non-B vs B-DNA TT curves to obtain pointwise and interval-wise adjusted p-values along the window, then aggregate them into a table of significant divergence per non-B type and per strand. Plot adjusted p-value trajectories and mean TT curve contrasts per type and strand with matplotlib/seaborn, saving iwt_results.csv and diagnostic figures in the results/ directory. The decision point is the interval-wise (not just pointwise) adjusted p-value, since interval-wise correction accounts for spatial correlation along the window.

## Related tools

- **IWTomics** (Interval-wise testing permutation framework used to obtain pointwise and interval-wise adjusted p-values along the TT curves)
- **pandas** (Loads and structures the per-window per-nucleotide TT dataset with window labels, strand, and position)
- **numpy** (Numerical backing for the IWT permutation test on TT curves)
- **matplotlib** (Plots adjusted p-value trajectories and mean TT curve contrasts per type and strand)
- **seaborn** (Diagnostic plotting of IWT results)

## Examples

```
import numpy, pandas; tt = pandas.read_csv(per_window_tt_table); iwt_pvals = iwt_intervalwise_test(tt, groups=['nonb_type','strand'], nperm=1000)
```

## Evaluation signals

- iwt_results.csv exists in results/ with rows split by non-B type and strand, containing both pointwise and interval-wise adjusted p-values
- The IWT permutation test flags significant TT divergence for most non-B DNA motif types on both forward and reverse strands, consistent with the reported NA12878 finding
- TT curves are aligned within windows of matching length per non-B type, and B-DNA controls are motif-free regions (Step 5) with positions extended to the opposite strand (Step 3)
- Diagnostic figures show adjusted p-value trajectories and mean TT curve contrasts for each non-B type and strand
- TT values trace back to the task_001 computation: time duration in seconds = number of current measurements / ONT sampling rate from Tombo re-squiggled reads

## Limitations

- IWT requires curves aligned within windows of matching length for each non-B type, so windows of differing lengths (e.g. 50 vs 100) must be tested separately
- Forward and reverse strands must be handled separately (positions extended to the opposite strand), so results may differ between strands
- Downstream validity depends on the upstream TT extraction: Albacore assumes a fixed block stride, whereas Tombo segmentation provides base-level current measurements needed for TT computation
- The experimental FAST5 data total over 4TB, making the upstream pipeline slow to rerun

## Evidence

- [abstract] Significant differences exist between DNA translocation timing for non-B DNA bases compared with B-DNA: "Based on whole genome nanopore sequencing of NA12878, we show that there exist significant differences between the timing of DNA translocation for non-B DNA bases compared with B-DNA."
- [readme] TT computation from Tombo-mapped reads: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
- [readme] Strand extension step: "Extend the positions on the opposite strand."
- [readme] Motif-free control windows: "Find motif free regions."
- [readme] FAST5 data size caveat: "Note: this may be slow as the FAST5 files total over 4TB"
