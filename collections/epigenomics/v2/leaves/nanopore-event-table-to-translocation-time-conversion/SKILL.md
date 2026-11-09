---
name: nanopore-event-table-to-translocation-time-conversion
description: Use when you have ONT FAST5 data basecalled with Albacore and need base-level
  translocation times in seconds — e.g. to build per-window TT quantiles for non-B
  DNA versus B-DNA comparisons.
when_to_use_negative:
- The input is already a per-base translocation-time table produced by Tombo re-squiggling
  — go directly to window preparation and TT quantile computation.
- 'You only have Albacore event tables without Tombo re-squiggling: Albacore assumes
  the block stride attribute remains fixed, so it cannot yield the base-level measurement
  counts needed for TT conversion.'
- Your goal is motif detection (GoFAE-DND novelty detection) rather than constructing
  the underlying TT signal.
tools:
- name: Albacore
  role: Basecalls raw ONT current into FASTQ/FAST5 with an event table that feeds
    re-squiggling
- name: Tombo
  role: Re-squiggles Albacore FAST5 output, segmenting raw current into base-level
    events with current measurement counts per position
  repo: https://nanoporetech.github.io/tombo/resquiggle.html
- name: h5py
  role: Reads FAST5 files needed for the novelty-detector / signal-processing steps
- name: pandas
  role: Loads and manipulates the per-base translocation-time dataset for downstream
    window quantile analysis
- name: numpy
  role: Supports numeric computation of translocation times and quantiles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:fc832030be2f7ba37a49300d7794e52f50e4dda5a8bd1ac63e3ea4079ff176ed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Albacore
  - Python
  - Tombo
  - h5py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# nanopore-event-table-to-translocation-time-conversion

## Summary

Converts Oxford Nanopore raw-signal events into per-base translocation times (seconds), computed as the number of current measurements per base divided by the ONT sampling rate. This translocation-time (TT) signal is the measurement basis for detecting non-B DNA structure effects in the ONT-nonb-GoFAE-DND workflow.

## When to use

Use when you have ONT FAST5 data basecalled with Albacore and need base-level translocation times in seconds — e.g. to build per-window TT quantiles for non-B DNA versus B-DNA comparisons.

## When NOT to use

- The input is already a per-base translocation-time table produced by Tombo re-squiggling — go directly to window preparation and TT quantile computation.
- You only have Albacore event tables without Tombo re-squiggling: Albacore assumes the block stride attribute remains fixed, so it cannot yield the base-level measurement counts needed for TT conversion.
- Your goal is motif detection (GoFAE-DND novelty detection) rather than constructing the underlying TT signal.

## Inputs

- ONT FAST5 files (NA12878 run SRR15058166, >4TB total)
- Albacore basecalled FAST5/FASTQ (event table, Basecall_1D_001 group)
- Reference genome FASTA (hg38.fa) for Tombo re-squiggling

## Outputs

- Per-base translocation-time table (seconds per base) from Tombo-mapped reads
- Per-read event segmentation (current measurements mapped to genomic positions) usable for window-level TT quantiles

## How to apply

Basecall the raw FAST5 current with Albacore (e.g. FLO-PRO002 / SQK-LSK109, r941_450bps_linear_prom.cfg) to produce FASTQ and FAST5 with an event table, then re-squiggle with Tombo against the hg38 reference ('tombo resquiggle ... --dna --basecall-group Basecall_1D_001 --include-event-stdev'). Albacore assumes a fixed block stride, so Tombo's segmentation must be used: it segments the raw current into per-base events. For each position on the Tombo-mapped reads, compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate. Load the resulting per-base TT table (e.g. with pandas/h5py) and hand it to downstream window steps (motif windows, motif-free B-DNA controls, 0.05–0.95 TT quantiles).

## Related tools

- **Albacore** (Basecalls raw ONT current into FASTQ/FAST5 with an event table that feeds re-squiggling)
- **Tombo** (Re-squiggles Albacore FAST5 output, segmenting raw current into base-level events with current measurement counts per position) — https://nanoporetech.github.io/tombo/resquiggle.html
- **h5py** (Reads FAST5 files needed for the novelty-detector / signal-processing steps)
- **pandas** (Loads and manipulates the per-base translocation-time dataset for downstream window quantile analysis)
- **numpy** (Supports numeric computation of translocation times and quantiles)

## Examples

```
read_fast5_basecaller.py -f FLO-PRO002 -k SQK-LSK109 --input $path/na12878/fast5/single/ --save_path $path/na12878/fast5/albacore_single/ --output_format fastq,fast5 -t 48 --recursive --config r941_450bps_linear_prom.cfg && tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --failed-reads-filename $path/workspace/pass/tombo_failed_reads.txt --processes 48; tt_seconds = n_current_measurements / ont_sampling_rate
```

## Evaluation signals

- Each TT value equals the per-base count of current measurements divided by the ONT sampling rate; for the r941_450bps configuration, per-base times should cluster near ~1/450 s.
- The output must be indexed per genomic position on Tombo-mapped reads so windows can be intersected with non-B DNA DB hg38 motif coordinates and motif-free B-DNA controls.
- Tombo re-squiggle should complete without unprocessed reads; the failed-reads file (tombo_failed_reads.txt) can be checked for coverage of the run.
- Downstream sanity check: TT quantiles (0.05, 0.25, 0.5, 0.75, 0.95) computed on 100 bp non-B motif windows must deviate from B-DNA windows with A-phased repeats, Z-DNA, G-quadruplex, and short tandem repeats showing the largest deviations (Fig. 4 pattern).
- Base-level event segmentation must come from Tombo, not raw Albacore strides — Albacore's fixed block stride would make TT values unusable.

## Limitations

- Requires the full 4TB+ FAST5 download of SRR15058166; the README warns 'this may be slow'.
- Albacore event tables alone are insufficient because they assume a fixed block stride; Tombo re-squiggling is mandatory.
- Tombo re-squiggling can fail on individual reads (failed reads written to tombo_failed_reads.txt), so read coverage is reduced.
- TT signal quality depends on the run chemistry/config (r941_450bps_linear_prom.cfg, FLO-PRO002 / SQK-LSK109); other chemistries have different sampling rates and speeds.

## Evidence

- [readme] For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate.: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
- [readme] The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride attribute remains fixed: "The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride attribute remains fixed"
- [readme] tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev: "tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev"
- [methods] Sequence bases are called from the raw ONT current using [Albacore]: "Sequence bases are called from the raw ONT current using [Albacore]"
