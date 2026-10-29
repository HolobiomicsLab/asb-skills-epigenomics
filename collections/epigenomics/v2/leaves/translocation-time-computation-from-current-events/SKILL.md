---
name: translocation-time-computation-from-current-events
description: Use when raw ONT FAST5 files (e.g. SRR15058166, R9.4.1 FLO-PRO002/SQK-LSK109)
  must be converted into per-position translocation-time signals for downstream non-B
  DNA structure modelling.
when_to_use_negative:
- Input is already basecalled FASTQ or a fixed-stride Albacore event table without
  per-base segmentation — Albacore alone assumes the block stride attribute remains
  fixed and cannot yield per-base translocation times.
- The goal is direct non-B motif annotation from sequence only, not translocation-time
  signal extraction (use the non-B DNA DB motif workflow instead).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Albacore
  role: Basecalls sequence bases from the raw ONT current and generates the FAST5/FASTQ
    output and initial event table
- name: Tombo
  role: Re-squiggles the Albacore FAST5 output against hg38, segmenting the raw signal
    into base-level events needed to compute translocation times
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: h5py
  role: Writes the per-base translocation-time matrix as translocation_times.h5
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:bc5ff87cfee004d1e1b23165421b609a02bbe965fa75f23e133bdc254638a0f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
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
yes_rate: 0.45454545454545453
n_claims: 11
---

# translocation-time-computation-from-current-events

## Summary

Computes per-base DNA translocation times (seconds) from Oxford Nanopore raw FAST5 current signals by basecalling with Albacore, re-squiggling with Tombo to obtain base-level event measurements, and dividing each event's measurement count by the ONT sampling rate.

## When to use

Use when raw ONT FAST5 files (e.g. SRR15058166, R9.4.1 FLO-PRO002/SQK-LSK109) must be converted into per-position translocation-time signals for downstream non-B DNA structure modelling.

## When NOT to use

- Input is already basecalled FASTQ or a fixed-stride Albacore event table without per-base segmentation — Albacore alone assumes the block stride attribute remains fixed and cannot yield per-base translocation times.
- The goal is direct non-B motif annotation from sequence only, not translocation-time signal extraction (use the non-B DNA DB motif workflow instead).

## Inputs

- Raw Oxford Nanopore FAST5 files (subset of SRR15058166, NA12878 whole-genome sequencing)
- hg38 reference genome FASTA (hg38.fa)

## Outputs

- Per-base translocation-time matrix keyed by genomic position, stored as translocation_times.h5 (h5py/HDF5)
- Tombo re-squiggled FAST5 files with base-level event measurements and event standard deviations
- tombo_failed_reads.txt log of reads failing re-squiggling

## How to apply

Basecall the raw current with Albacore's read_fast5_basecaller.py using flowcell FLO-PRO002, kit SQK-LSK109, config r941_450bps_linear_prom.cfg, output_format fastq,fast5, 48 threads, saving to an albacore_single workspace. Then re-squiggle the Albacore FAST5 output with Tombo against hg38 (tombo resquiggle ... --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --processes 48), which segments the raw signal into base-level events and corrects spurious variation using the reference genome — Tombo is used instead of Albacore because Albacore's fixed block stride attribute does not yield per-base current measurements. For each position on the Tombo-mapped reads, compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate. Finally assemble the per-base translocation times into a matrix keyed by genomic position and write it as translocation_times.h5 with h5py.

## Related tools

- **Albacore** (Basecalls sequence bases from the raw ONT current and generates the FAST5/FASTQ output and initial event table)
- **Tombo** (Re-squiggles the Albacore FAST5 output against hg38, segmenting the raw signal into base-level events needed to compute translocation times) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **h5py** (Writes the per-base translocation-time matrix as translocation_times.h5) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND

## Examples

```
tombo resquiggle $path/workspace/pass/ hg38.fa --dna --overwrite --basecall-group Basecall_1D_001 --include-event-stdev --failed-reads-filename $path/workspace/pass/tombo_failed_reads.txt --processes 48
```

## Evaluation signals

- Tombo re-squiggle completes and produces per-base event-level current measurements for passing reads, with failed reads logged to tombo_failed_reads.txt.
- The output translocation_times.h5 exists and contains a per-base translocation time matrix keyed by genomic position.
- Each computed duration equals the ratio of the event's number of current measurements to the ONT sampling rate (spot-check a sample of events).
- Basecalling command parameters match the R9.4.1 protocol: FLO-PRO002 flowcell, SQK-LSK109 kit, r941_450bps_linear_prom.cfg, Basecall_1D_001 group passed to Tombo.

## Limitations

- The full FAST5 dataset totals over 4TB, so downloading is slow; a download.sh script is provided to download, unpack, and clean up a subset.
- Some reads fail Tombo re-squiggling and must be tracked via the --failed-reads-filename log rather than silently dropped.
- Albacore's event table alone assumes a fixed block stride attribute and cannot provide the base-level segmentation required for translocation times.

## Evidence

- [methods] Sequence bases are called from the raw ONT current using [Albacore]: [evidence span withheld — anchor status: not_found]
- [readme] the re-squiggling algorithm segments the raw current signal into events and calls nucleotide bases using the current and a reference genome: "the re-squiggling algorithm segments the raw current signal into events and calls nucleotide bases using the current and a reference genome for"
- [methods] For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate.: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
- [readme] The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride attribute remains fixed: "The Tombo segmentation provides current measurements at the base-level, unlike Albacore, which assumes the block stride
attribute remains fixed, which enables the computation of translocation times"
