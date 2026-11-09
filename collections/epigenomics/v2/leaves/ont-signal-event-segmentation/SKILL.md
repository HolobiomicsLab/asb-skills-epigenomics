---
name: ont-signal-event-segmentation
description: Use when raw ONT FAST5 current signals must be converted into per-base
  translocation times (seconds per genomic position) for sequencing-speed or non-B
  DNA structure analyses.
when_to_use_negative:
- Input is already basecalled FASTQ or a pre-computed event table with fixed block
  strides — Albacore's fixed stride assumption is precisely what Tombo re-squiggling
  is meant to overcome
- The analysis targets base modifications detected directly from current signal rather
  than translocation timing
- No reference genome is available, since Tombo re-squiggling requires a reference
  (hg38.fa) to correct spurious variation and segment events
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Albacore
  role: Basecalls raw ONT current into sequence and FAST5 output with an event table
    (read_fast5_basecaller.py, flowcell FLO-PRO002, kit SQK-LSK109, config r941_450bps_linear_prom.cfg)
- name: Tombo
  role: Re-squiggles the Albacore FAST5 output against hg38, segmenting the raw current
    signal into base-level events needed for translocation-time computation
  repo: https://nanoporetech.github.io/tombo/resquiggle.html
- name: h5py
  role: Python library used to write the per-base translocation time matrix as an
    HDF5 file (translocation_times.h5)
- name: numpy
  role: Numerical computation for per-position translocation time ratios
- name: pandas
  role: Assembly of per-base translocation times into a position-keyed matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:1facdbc9850c66ce9fe39007c5e66364bb3936519409aec6d6ed073087d26449
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
yes_rate: 0.5
n_claims: 10
---

# ont-signal-event-segmentation

## Summary

Reconstructs per-base translocation times from raw Oxford Nanopore FAST5 current signals by basecalling with Albacore and re-squiggling with Tombo, then computing each position's duration as the ratio of current measurements to the ONT sampling rate. This converts raw signal into per-position translocation times suitable for downstream non-B DNA modelling.

## When to use

Use when raw ONT FAST5 current signals must be converted into per-base translocation times (seconds per genomic position) for sequencing-speed or non-B DNA structure analyses.

## When NOT to use

- Input is already basecalled FASTQ or a pre-computed event table with fixed block strides — Albacore's fixed stride assumption is precisely what Tombo re-squiggling is meant to overcome
- The analysis targets base modifications detected directly from current signal rather than translocation timing
- No reference genome is available, since Tombo re-squiggling requires a reference (hg38.fa) to correct spurious variation and segment events

## Inputs

- Raw Oxford Nanopore FAST5 files (subset of SRR15058166, NA12878 whole-genome nanopore sequencing)
- hg38 reference genome FASTA (hg38.fa)
- download.sh script from the bayesomicslab/ONT-nonb-GoFAE-DND repository

## Outputs

- Tombo re-squiggled FAST5 files with per-base event-level current measurements
- tombo_failed_reads.txt log of reads that failed re-squiggling
- Per-base translocation time matrix keyed by genomic position, stored as translocation_times.h5 (h5py/HDF5)

## How to apply

Download a raw FAST5 subset from the SRR15058166 deposit (total data exceed 4TB, so use the provided download.sh script). Basecall the raw current with Albacore's read_fast5_basecaller.py using flowcell FLO-PRO002, kit SQK-LSK109, and config r941_450bps_linear_prom.cfg with output_format fastq,fast5 and 48 threads. Re-squiggle the Albacore FAST5 output with Tombo against the hg38 reference (--dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --processes 48), which segments the raw signal into base-level events and corrects spurious variation using the reference — unlike Albacore, which assumes a fixed block stride attribute. For each position on the Tombo-mapped reads, compute the time duration in seconds as the number of current measurements divided by the ONT sampling rate, then assemble per-base translocation times into a matrix keyed by genomic position (written as translocation_times.h5 with h5py).

## Related tools

- **Albacore** (Basecalls raw ONT current into sequence and FAST5 output with an event table (read_fast5_basecaller.py, flowcell FLO-PRO002, kit SQK-LSK109, config r941_450bps_linear_prom.cfg))
- **Tombo** (Re-squiggles the Albacore FAST5 output against hg38, segmenting the raw current signal into base-level events needed for translocation-time computation) — https://nanoporetech.github.io/tombo/resquiggle.html
- **h5py** (Python library used to write the per-base translocation time matrix as an HDF5 file (translocation_times.h5))
- **numpy** (Numerical computation for per-position translocation time ratios)
- **pandas** (Assembly of per-base translocation times into a position-keyed matrix)

## Examples

```
read_fast5_basecaller.py -f FLO-PRO002 -k SQK-LSK109 --input $path/na12878/fast5/single/ --save_path $path/na12878/fast5/albacore_single/ --output_format fastq,fast5 -t 48 --recursive --config r941_450bps_linear_prom.cfg; tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --failed-reads-filename $path/workspace/pass/tombo_failed_reads.txt --processes 48
```

## Evaluation signals

- Tombo re-squiggle completes with per-base event-level current measurements produced for passing reads
- Failed reads are logged to tombo_failed_reads.txt rather than silently dropped
- The output HDF5 file translocation_times.h5 exists and contains a per-base translocation time matrix
- Each computed duration equals the ratio of the event's measurement count to the ONT sampling rate
- Events are base-level (variable stride) rather than the fixed block strides produced by Albacore alone

## Limitations

- The full FAST5 dataset totals over 4TB, so downloading can be slow; the download.sh script mitigates this by downloading, unpacking, and cleaning up incrementally
- Some reads fail Tombo re-squiggling and are excluded (logged in tombo_failed_reads.txt), reducing coverage
- Albacore alone cannot yield translocation times because it assumes the block stride attribute remains fixed; Tombo segmentation is required

## Evidence

- [readme] Sequence bases are called from the raw ONT current using [Albacore](http://porecamp.github.io/2017/basecalling.html), which generates an event table that describes the DNA context in the nanopore: "Sequence bases are called from the raw ONT current using [Albacore](http://porecamp.github.io/2017/basecalling.html), which generates an event table that describes the DNA context"
- [readme] the re-squiggling algorithm segments the raw current signal into events and calls nucleotide bases using the current and a reference genome for correcting spurious variation: "the re-squiggling algorithm segments the raw current signal into events and calls nucleotide bases using the current and a reference genome"
- [readme] For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate.: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements"
- [readme] The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride attribute remains fixed: "The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride"
- [readme] Note: this may be slow as the FAST5 files total over 4TB: "Note: this may be slow as the FAST5 files total over 4TB"
