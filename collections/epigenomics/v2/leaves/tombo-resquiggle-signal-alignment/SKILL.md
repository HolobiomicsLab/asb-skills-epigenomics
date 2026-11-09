---
name: tombo-resquiggle-signal-alignment
description: Use when raw nanopore FAST5 signals must be aligned to a reference at
  base-event resolution to extract per-position translocation times, because Albacore's
  fixed block strides are insufficient for timing analysis.
when_to_use_negative:
- Input is already basecalled FASTQ or an event table without raw signal — re-squiggling
  requires raw FAST5 current data.
- The analysis only needs sequence calls, not per-base timing — Albacore's event table
  with fixed block strides suffices.
- No reference genome is available for the organism — Tombo re-squiggling is reference-guided
  and cannot proceed without a FASTA.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Albacore
  role: Basecalls sequence bases from the raw ONT current and produces the FAST5 output
    with Basecall_1D_001 group that Tombo re-squiggles against
- name: Tombo
  role: Re-squiggles the Albacore FAST5 output, segmenting raw current into base-level
    events and correcting spurious variation against hg38
  repo: https://nanoporetech.github.io/tombo/resquiggle.html
- name: h5py
  role: Writes the assembled per-base translocation time matrix as translocation_times.h5
- name: numpy
  role: Supports computation of per-position translocation times from measurement
    counts and sampling rate
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:91b24d49c47aaa7d27e1e70fd6976c67923e16d684117cf41141a14a69104923
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Albacore
  - Python
  - Tombo
  - h5py
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# tombo-resquiggle-signal-alignment

## Summary

Re-squiggle Albacore-basecalled ONT FAST5 reads against the hg38 reference with Tombo, segmenting raw current into base-level events so per-base translocation times can be computed as measurement counts divided by the sampling rate.

## When to use

Use when raw nanopore FAST5 signals must be aligned to a reference at base-event resolution to extract per-position translocation times, because Albacore's fixed block strides are insufficient for timing analysis.

## When NOT to use

- Input is already basecalled FASTQ or an event table without raw signal — re-squiggling requires raw FAST5 current data.
- The analysis only needs sequence calls, not per-base timing — Albacore's event table with fixed block strides suffices.
- No reference genome is available for the organism — Tombo re-squiggling is reference-guided and cannot proceed without a FASTA.

## Inputs

- Raw ONT FAST5 files (subset of SRR15058166 NA12878 whole-genome nanopore run)
- Albacore basecalled FAST5 output in workspace/pass/ (Basecall_1D_001 group)
- hg38 reference genome FASTA (hg38.fa)

## Outputs

- Tombo re-squiggled FAST5 files with per-base event-level current measurements (including event standard deviations)
- tombo_failed_reads.txt log of reads that failed re-squiggling
- Per-base translocation time matrix keyed by genomic position, saved as translocation_times.h5

## How to apply

First basecall raw current with Albacore (read_fast5_basecaller.py, flowcell FLO-PRO002, kit SQK-LSK109, config r941_450bps_linear_prom.cfg, output_format fastq,fast5). Then run Tombo resquiggle on the pass/ FAST5 output against hg38.fa with --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev, logging failures with --failed-reads-filename, so the raw signal is segmented into base-level events and spurious variation is corrected against the reference. The rationale is that Tombo's segmentation yields variable-length current measurement groups per base, unlike Albacore's fixed block stride, enabling each position's duration in seconds to be computed as the ratio of the number of current measurements to the ONT sampling rate. Assemble the resulting per-base translocation times into a matrix keyed by genomic position and write it with h5py as translocation_times.h5.

## Related tools

- **Albacore** (Basecalls sequence bases from the raw ONT current and produces the FAST5 output with Basecall_1D_001 group that Tombo re-squiggles against)
- **Tombo** (Re-squiggles the Albacore FAST5 output, segmenting raw current into base-level events and correcting spurious variation against hg38) — https://nanoporetech.github.io/tombo/resquiggle.html
- **h5py** (Writes the assembled per-base translocation time matrix as translocation_times.h5)
- **numpy** (Supports computation of per-position translocation times from measurement counts and sampling rate)

## Examples

```
tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --failed-reads-filename $path/workspace/pass/tombo_failed_reads.txt --processes 48
```

## Evaluation signals

- Tombo re-squiggle completes and produces per-base event-level current measurements for passing reads, with failures logged to tombo_failed_reads.txt
- The output HDF5 file exists and contains a per-base translocation time matrix keyed by genomic position
- Each computed duration equals the ratio of the event's number of current measurements to the ONT sampling rate
- Re-squiggled events are anchored to the hg38 reference via the Basecall_1D_001 basecall group

## Limitations

- Tombo re-squiggling fails on a subset of reads; failed reads must be logged and excluded (tombo_failed_reads.txt)
- Albacore's event table assumes the block stride attribute remains fixed, so it cannot directly support per-base translocation-time computation — re-squiggling is required
- Full SRR15058166 FAST5 data exceeds 4TB, so downloads may be slow and a subset with download.sh is recommended

## Evidence

- [readme] Sequence bases are called from the raw ONT current using Albacore, which generates an event table that describes the DNA context in the nanopore: "Sequence bases are called from the raw ONT current using [Albacore](http://porecamp.github.io/2017/basecalling.html), which generates an event table that describes the DNA context in the nanopore"
- [methods] The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride attribute remains fixed, which enables the computation of translocation times: "The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride attribute remains fixed, which enables the computation of translocation times"
- [readme] For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate.: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
- [readme] $ tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev: "$ tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev"
