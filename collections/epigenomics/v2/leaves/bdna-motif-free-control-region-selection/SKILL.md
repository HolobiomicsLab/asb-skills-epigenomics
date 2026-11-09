---
name: bdna-motif-free-control-region-selection
description: 'Use when reproducing the Fig. 4 TT-quantile deviation analysis and you
  need a matched B-DNA control set: motif positions are loaded from non-B DNA DB hg38
  and you must exclude all motif-overlapping windows before computing per-window translocation-time
  quantiles.'
when_to_use_negative:
- The genomic windows of interest already overlap annotated non-B motifs — these are
  the case windows, not controls.
- You are running the novelty-detection or classifier stage (GoFAE-DND, isolation
  forest, one-class SVM) on simulated data, where the simulator supplies B-DNA windows
  directly and no motif-free region scan is needed.
- Input is already a per-window quantile table; the control regions must exist upstream
  before quantiles can be compared.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: non-B DNA DB
  role: Source of hg38 motif positions used to define which regions are motif-free
    and must be excluded from B-DNA controls
  repo: https://nonb-abcc.ncifcrf.gov/apps/site/default
- name: Tombo
  role: Re-squiggles Albacore-basecalled FAST5 reads against hg38.fa to obtain base-level
    current measurements from which translocation times for reads on candidate control
    windows are computed
  repo: https://nanoporetech.github.io/tombo/tutorials.html
- name: pandas
  role: Loads the extracted per-base translocation-time dataset and restricts it to
    non-overlapping motif-free control windows
- name: numpy
  role: Computes the 0.05, 0.25, 0.5, 0.75, 0.95 translocation-time quantiles per
    B-DNA control window
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:8496371255c2b66f9b1b26797acf42f64858f7abe8aaec9abe36933d7b3beaee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Tombo
  - matplotlib
  - non-B DNA DB
  - numpy
  - pandas
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# bdna-motif-free-control-region-selection

## Summary

Selects B-DNA control windows from motif-free regions of the hg38 reference that overlap high-quality ONT reads, as in Step 5 of the paper's window-preparation pipeline, to provide the baseline translocation-time distribution against which non-B DNA motif windows (100 bp, centered on non-B DNA DB motifs) are compared.

## When to use

Use when reproducing the Fig. 4 TT-quantile deviation analysis and you need a matched B-DNA control set: motif positions are loaded from non-B DNA DB hg38 and you must exclude all motif-overlapping windows before computing per-window translocation-time quantiles.

## When NOT to use

- The genomic windows of interest already overlap annotated non-B motifs — these are the case windows, not controls.
- You are running the novelty-detection or classifier stage (GoFAE-DND, isolation forest, one-class SVM) on simulated data, where the simulator supplies B-DNA windows directly and no motif-free region scan is needed.
- Input is already a per-window quantile table; the control regions must exist upstream before quantiles can be compared.

## Inputs

- non-B DNA DB hg38 motif position annotations (all structure classes, e.g. G-quadruplex, Z-DNA, short tandem repeat, A-phased repeat)
- hg38 reference FASTA (hg38.fa)
- Tombo re-squiggled per-base translocation-time events from NA12878 (SRR15058166) FAST5 reads, with time in seconds = number of current measurements / ONT sampling rate
- High-quality read set from Step 4 (reads falling on candidate windows)
- window_size parameter (100 for experimental dataset; 50 and 100 also evaluated)

## Outputs

- Set of B-DNA control windows from motif-free regions overlapping high-quality reads
- Per-window translocation-time quantiles (0.05, 0.25, 0.5, 0.75, 0.95) for the B-DNA control distribution
- CSV table of per-type quantile and deviation results with diagnostic summary statistics in the root-level results/ directory

## How to apply

After fixing windows of length x (window_size = 100 for the experimental dataset) centered on each non-B DNA DB motif and extending positions to the opposite strand (Steps 1–3), identify genomic regions free of any annotated non-B motif from the non-B DNA DB hg38 annotations ('Find motif free regions', Step 5). Restrict candidate control windows to those falling on high-quality reads found in Step 4 (reads that pass Tombo re-squiggle of the Albacore-basecalled SRR15058166 FAST5 data). Compute the translocation signal on non-overlapping control windows (Step 6), converting current measurements to seconds as number of current measurements divided by the ONT sampling rate. These B-DNA control windows then supply the reference distribution of 0.05, 0.25, 0.5, 0.75, and 0.95 translocation-time quantiles from which each non-B type's deviation is quantified; ensure controls and non-B windows are non-overlapping so the comparison is uncontaminated.

## Related tools

- **non-B DNA DB** (Source of hg38 motif positions used to define which regions are motif-free and must be excluded from B-DNA controls) — https://nonb-abcc.ncifcrf.gov/apps/site/default
- **Tombo** (Re-squiggles Albacore-basecalled FAST5 reads against hg38.fa to obtain base-level current measurements from which translocation times for reads on candidate control windows are computed) — https://nanoporetech.github.io/tombo/tutorials.html
- **pandas** (Loads the extracted per-base translocation-time dataset and restricts it to non-overlapping motif-free control windows)
- **numpy** (Computes the 0.05, 0.25, 0.5, 0.75, 0.95 translocation-time quantiles per B-DNA control window)

## Examples

```
datasets = ['experimental', 'simulated']; window_size = [50, 100]
```

## Evaluation signals

- No selected B-DNA control window overlaps any non-B DNA DB motif position on either strand (after Step 3 opposite-strand extension).
- Control windows and non-B windows are non-overlapping, per Step 6 of the window preparation.
- Every retained control window overlaps at least one high-quality read from the Step 4 read set.
- Per-window quantile fields (0.05, 0.25, 0.5, 0.75, 0.95) exist in the output CSV alongside deviation fields and diagnostic summary statistics.
- Downstream Fig. 4 reproduction shows significant deviation of non-B types from the B-DNA control distribution, with A-phased repeats, Z-DNA, G-quadruplex, and short tandem repeats deviating most.

## Limitations

- Non-B DNA motifs are necessary but not sufficient indicators of non-B structures, so motif-free regions may still harbor unannotated non-B DNA.
- The FAST5 data for SRR15058166 total over 4TB, making download and the read-quality filter step slow.
- Some reads fail Tombo re-squiggling (failed reads are written to tombo_failed_reads.txt) and must be excluded from the high-quality read set.

## Evidence

- [methods] Step 5: Find motif free regions.: "Step 5: Find motif free regions."
- [methods] Find high quality reads that fall on the windows.: "Find high quality reads that fall on the windows."
- [readme] Find motif free regions.: "Find motif free regions."
- [methods] Step 6: Compute translocation signal on the non-overlapping windows.: "Step 6: Compute translocation signal on the non-overlapping windows."
- [methods] Extract motifs postions from non-B DNA DB: "Extract motifs postions from [non-B DNA DB]"
