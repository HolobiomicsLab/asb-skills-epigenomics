---
name: non-b-dna-motif-window-annotation-hg38
description: Use when ONT re-squiggled per-base translocation times must be assigned
  to genomic windows — 100 bp windows centered on non-B DNA DB hg38 motifs (G-quadruplex,
  Z-DNA, STR, A-phased repeat, etc.) plus motif-free B-DNA controls — for TT quantile
  comparison.
when_to_use_negative:
- 'For simulated data: the bundled simulator (simulator.py -nb 10000 -b 1000000) already
  generates non-B and B-DNA windows directly, so motif-based window preparation on
  hg38 is unnecessary'
- When the goal is GoFAE-DND novelty-detection model training (Main.py) rather than
  descriptive TT-quantile deviation analysis of annotated windows
- When the input is already a per-window quantile/deviation table
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: pandas
  role: Loads the extracted translocation-time dataset and motif annotations, and
    computes per-window TT quantiles on non-overlapping windows
- name: numpy
  role: Numeric backend for quantile computation and deviation aggregation by non-B
    structure class
- name: matplotlib
  role: Generates diagnostic plots alongside the quantile/deviation CSV in results/
- name: seaborn
  role: Statistical plotting of TT quantile deviations per non-B type versus B-DNA
    controls
- name: Tombo
  role: Re-squiggles Albacore FAST5 output to provide base-level current measurements
    from which per-base translocation times are computed and matched to windows
  repo: https://nanoporetech.github.io/tombo/tutorials.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:dde17606ce2ee2f0b66513dfce4ceed2f84cc405745444e3e0a1d315298d0706
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Tombo
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# non-b-dna-motif-window-annotation-hg38

## Summary

Annotates the NA12878 ONT translocation-time dataset with 100 bp windows centered on non-B DNA DB (hg38) motifs plus motif-free B-DNA control windows, enabling per-non-B-type translocation-time (TT) quantile comparison against B-DNA as in the paper's Fig. 4 deviation analysis.

## When to use

Use when ONT re-squiggled per-base translocation times must be assigned to genomic windows — 100 bp windows centered on non-B DNA DB hg38 motifs (G-quadruplex, Z-DNA, STR, A-phased repeat, etc.) plus motif-free B-DNA controls — for TT quantile comparison.

## When NOT to use

- For simulated data: the bundled simulator (simulator.py -nb 10000 -b 1000000) already generates non-B and B-DNA windows directly, so motif-based window preparation on hg38 is unnecessary
- When the goal is GoFAE-DND novelty-detection model training (Main.py) rather than descriptive TT-quantile deviation analysis of annotated windows
- When the input is already a per-window quantile/deviation table

## Inputs

- Per-base translocation-time table from Tombo re-squiggle events of NA12878 FAST5 files (SRR15058166), time in seconds = number of current measurements / ONT sampling rate
- Non-B DNA DB hg38 motif position annotations (G-quadruplex, Z-DNA, short tandem repeat, A-phased repeat, etc.)
- hg38 reference genome FASTA used for Tombo re-squiggle mapping
- window_size parameter (100 for the experimental dataset; 50 also evaluated)

## Outputs

- Per-window annotated table of non-B type, B-DNA control label, and TT quantiles (0.05, 0.25, 0.5, 0.75, 0.95)
- Per-non-B-type deviation results versus the B-DNA control distribution with diagnostic summary statistics
- CSV table and matplotlib/seaborn diagnostic plots in the root-level results/ directory

## How to apply

Extract motif positions from the non-B DNA DB (hg38) and fix windows of length x around each motif (window_size=100 for the experimental dataset; the paper evaluates both 50 and 100), extending positions to the opposite strand per the paper's Step 3. Select B-DNA control windows from motif-free regions (Step 5) that overlap high-quality reads falling on the windows (Step 4), where the per-base events come from Tombo re-squiggle of Albacore-basecalled FAST5 files (basecall-group Basecall_1D_001, --include-event-stdev). Compute per-base translocation time in seconds as the ratio of the number of current measurements to the ONT sampling rate, restrict to non-overlapping windows (Step 6), then compute the 0.05, 0.25, 0.5, 0.75, and 0.95 TT quantiles per window for each non-B type and the B-DNA controls using numpy/pandas. Aggregate quantile deviations from the B-DNA control distribution by non-B structure class and write per-type quantile/deviation tables plus matplotlib/seaborn diagnostics to the root-level results/ directory.

## Related tools

- **pandas** (Loads the extracted translocation-time dataset and motif annotations, and computes per-window TT quantiles on non-overlapping windows)
- **numpy** (Numeric backend for quantile computation and deviation aggregation by non-B structure class)
- **matplotlib** (Generates diagnostic plots alongside the quantile/deviation CSV in results/)
- **seaborn** (Statistical plotting of TT quantile deviations per non-B type versus B-DNA controls)
- **Tombo** (Re-squiggles Albacore FAST5 output to provide base-level current measurements from which per-base translocation times are computed and matched to windows) — https://nanoporetech.github.io/tombo/tutorials.html

## Examples

```
qs = df.groupby('window_id')['time_s'].quantile([0.05,0.25,0.5,0.75,0.95]).unstack()
```

## Evaluation signals

- Output CSV exists in results/ with required quantile (0.05–0.95) and deviation fields per non-B type and B-DNA control
- Windows are 100 bp and centered on non-B DNA DB motifs, with opposite-strand positions extended (Step 3) and non-overlapping windows used (Step 6)
- B-DNA controls derive from motif-free regions overlapping high-quality reads, with no overlap with annotated non-B motif windows
- Reproduced deviation ranking matches Fig. 4: A-phased repeats, Z-DNA, G-quadruplex, and short tandem repeats show the largest significant TT-quantile deviations from B-DNA
- Per-base times are in seconds (measurements / ONT sampling rate), and quantiles are finite and positive across all non-B types

## Limitations

- Non-B DNA motifs are necessary but not sufficient indicators of actual non-B structures, so motif-annotated windows may not all adopt non-B conformations
- Experimental FAST5 data total over 4TB and the download may be slow; Tombo re-squiggle fails on some reads, which must be tracked via --failed-reads-filename
- Requiring non-overlapping windows and motif-free controls discards many motif instances and can reduce per-type sample sizes

## Evidence

- [readme] Extract motifs postions from non-B DNA DB hg38: "Extract motifs postions from [non-B DNA DB](https://nonb-abcc.ncifcrf.gov/apps/site/default)."
- [readme] Window sizing and opposite-strand extension steps: "Step 2: 
Fix windows of length x around motifs.

Step 3: 
Extend the positions on the opposite strand."
- [readme] Motif-free B-DNA control and non-overlapping window steps: "Step 5:
Find motif free regions.

Step 6:
Compute translocation signal on the non-overlapping windows."
- [methods] Translocation time computation from current measurements: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
- [results] Window sizes 50 and 100 for experimental dataset: "datasets = ['experimental', 'simulated']
window_size = [50, 100]"
- [methods] High-quality read overlap selection for windows: "Step 4: Find high quality reads that fall on the windows."
