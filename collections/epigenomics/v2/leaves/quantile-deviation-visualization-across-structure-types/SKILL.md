---
name: quantile-deviation-visualization-across-structure-types
description: Use when you have per-base translocation times from Tombo re-squiggled
  ONT reads and motif coordinates (e.g. non-B DNA DB hg38 annotations), and need to
  test which non-B structure classes show significant TT quantile deviations from
  B-DNA.
when_to_use_negative:
- Input is simulated translocation-time data at window_size = 50 rather than the experimental
  dataset at window_size = 100 — the window-fixing and control-selection steps differ
- The goal is non-B structure classification/prediction with GoFAE-DND rather than
  descriptive quantile-deviation comparison against B-DNA controls
- Motif coordinates come from a genome assembly other than hg38 or from motifs not
  in the non-B DNA DB, so windows cannot be matched to the Tombo-mapped reads
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: pandas
  role: Load the per-base TT dataset, build 100 bp windows around non-B DNA DB hg38
    motifs, and compute per-window TT quantiles
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Compute the 0.05–0.95 TT quantiles and deviation statistics per window and
    structure type
- name: matplotlib
  role: Generate the Fig. 4-style diagnostic deviation plots written to the root-level
    results/ directory
- name: seaborn
  role: Statistical visualization of TT quantile deviations by non-B structure class
- name: Tombo
  role: Upstream re-squiggling that provides base-level current measurements from
    which translocation times are derived
  repo: https://nanoporetech.github.io/tombo/tutorials.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:942912ce98520ff65894a9d6ca52d631e541c5c778bdf804c07e41f809c9fbe8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - Tombo
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# quantile-deviation-visualization-across-structure-types

## Summary

Reproduces the Fig. 4 analysis by computing per-window TT (translocation time) quantiles (0.05, 0.25, 0.5, 0.75, 0.95) for 100 bp windows centred on non-B DNA motifs versus motif-free B-DNA controls, then quantifying and plotting the deviation of each non-B type from the B-DNA distribution.

## When to use

Use when you have per-base translocation times from Tombo re-squiggled ONT reads and motif coordinates (e.g. non-B DNA DB hg38 annotations), and need to test which non-B structure classes show significant TT quantile deviations from B-DNA.

## When NOT to use

- Input is simulated translocation-time data at window_size = 50 rather than the experimental dataset at window_size = 100 — the window-fixing and control-selection steps differ
- The goal is non-B structure classification/prediction with GoFAE-DND rather than descriptive quantile-deviation comparison against B-DNA controls
- Motif coordinates come from a genome assembly other than hg38 or from motifs not in the non-B DNA DB, so windows cannot be matched to the Tombo-mapped reads

## Inputs

- Per-base translocation-time dataset from Tombo re-squiggle events (time in seconds = number of current measurements / ONT sampling rate)
- non-B DNA DB hg38 motif annotations (G-quadruplex, Z-DNA, short tandem repeat, A-phased repeat, etc.)
- Motif-free (B-DNA) control regions overlapping high-quality reads
- Window size parameter (window_size = 100 for the experimental dataset)

## Outputs

- CSV table of per-type TT quantiles (0.05, 0.25, 0.5, 0.75, 0.95) and deviations from B-DNA, with diagnostic summary statistics, in the root-level results/ directory
- Diagnostic quantile-deviation plots (Fig. 4 style) generated with matplotlib/seaborn

## How to apply

Load the per-base translocation-time dataset (Tombo events converted to seconds as number of current measurements divided by the ONT sampling rate) with pandas. Fix 100 bp windows centred on each non-B DNA DB hg38 motif (window_size = 100 for the experimental dataset), extend positions to the opposite strand, and select B-DNA control windows from motif-free regions overlapping high-quality reads (Steps 2, 3, and 5 of the window preparation). Restrict the TT data to non-overlapping windows and compute the 0.05, 0.25, 0.5, 0.75, and 0.95 TT quantiles per window for each non-B type (G-quadruplex, Z-DNA, short tandem repeat, A-phased repeat, etc.) and for B-DNA controls using numpy/pandas. Aggregate deviations by non-B structure class, apply the significance assessment used for Fig. 4, and write per-type quantile/deviation results plus diagnostic plots (matplotlib/seaborn) to a root-level results/ directory.

## Related tools

- **pandas** (Load the per-base TT dataset, build 100 bp windows around non-B DNA DB hg38 motifs, and compute per-window TT quantiles) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Compute the 0.05–0.95 TT quantiles and deviation statistics per window and structure type)
- **matplotlib** (Generate the Fig. 4-style diagnostic deviation plots written to the root-level results/ directory)
- **seaborn** (Statistical visualization of TT quantile deviations by non-B structure class)
- **Tombo** (Upstream re-squiggling that provides base-level current measurements from which translocation times are derived) — https://nanoporetech.github.io/tombo/tutorials.html

## Examples

```
df = pd.read_csv('tt_events.csv'); q = df.groupby(['window','nonb_type'])['tt_sec'].quantile([0.05,0.25,0.5,0.75,0.95])
```

## Evaluation signals

- Output CSV exists in results/ and contains the required fields: the five TT quantiles (0.05, 0.25, 0.5, 0.75, 0.95) and per-type deviation statistics for every non-B structure class plus B-DNA controls
- Reproduced deviation ranking matches the Fig. 4 pattern: all non-B types deviate significantly from B-DNA, with A-phased repeats, Z-DNA, G-quadruplex, and short tandem repeats showing the largest deviations across all TT quantiles
- Windows are non-overlapping and centred on motifs with opposite-strand positions extended, and B-DNA controls come from motif-free regions (Steps 2, 3, 5 of window preparation)
- TT values are in seconds computed as number of current measurements divided by the ONT sampling rate
- Diagnostic plots in results/ render one deviation profile per non-B type comparable to the published Fig. 4

## Limitations

- Non-B DNA base motifs are necessary but not sufficient indicators of non-B structures, so a significant TT deviation indicates association with motif regions, not confirmed structural formation
- The experimental analysis is tied to window_size = 100 (with 50 used for the simulated dataset); using other window sizes changes the quantile-deviation results
- Analysis depends on Tombo re-squiggle success; reads that fail re-squiggling (tracked in tombo_failed_reads.txt) are excluded, and the FAST5 input totals over 4TB, making full reproduction slow

## Evidence

- [readme] we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
- [methods] Step 2: Fix windows of length x around motifs.: [evidence span withheld — anchor status: not_found]
- [results] datasets = ['experimental', 'simulated'] window_size = [50, 100]: "datasets = ['experimental', 'simulated']
window_size = [50, 100]"
- [readme] computational methods rely on non-B DNA base motifs, which are necessary but not sufficient indicators of non-B structures: "while computational methods rely on non-B DNA base motifs, which are necessary but not sufficient indicators of non-B structures"
