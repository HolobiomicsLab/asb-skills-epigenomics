---
name: nanopore-raw-signal-basecalling
description: Use when you have raw nanopore FAST5 electrolytic-current signal files
  and need to decode them into per-read nucleotide sequences without prior signal
  segmentation.
when_to_use_negative:
- 'Your genome is large: Chiron runs at only ~2000 bp/s, making it unsuitable for
  large genomes'
- Your input is already base-called (FASTQ/BAM) rather than raw FAST5 signal — re-base-calling
  is unnecessary
- You need a production ONT-supported caller with modified-base support rather than
  an end-to-end research/benchmarking tool
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: 'End-to-end deep-learning base-caller: decodes raw FAST5 ion-current segments
    into nucleotide sequences via a CTC-trained model, outputting FASTQ'
  repo: https://github.com/haotianteng/chiron
- name: Oxford Nanopore
  role: Sequencing platform producing the raw FAST5 electrolytic-current signal files
    that serve as input to the base-caller
- name: Taiyaki
  role: Related ONT research software for training flip-flop base-calling models used
    by Guppy; illustrates the CTC-inspired training paradigm underlying end-to-end
    base-callers
  repo: https://github.com/nanoporetech/taiyaki
- name: Bonito
  role: Open source ONT research basecaller alternative for decoding Oxford Nanopore
    reads from signal to FASTQ/BAM
  repo: https://github.com/nanoporetech/bonito
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:cbdc3e1e09ba8c1c8741b4894310e000856b8fca8041fd4610c66ff10c104c8c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jts/nanopore-paper-analysis
  repo_url: https://github.com/haotianteng/chiron
  tools:
  - Bonito
  - Chiron
  - Oxford Nanopore
  - Taiyaki
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-raw-signal-basecalling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Base-call Oxford Nanopore raw ion-current signal (FAST5) directly into nucleotide sequences using an end-to-end deep-learning model such as Chiron, which was the first deep-learning model to translate raw signal to sequence without an error-prone segmentation step.

## When to use

Use when you have raw nanopore FAST5 electrolytic-current signal files and need to decode them into per-read nucleotide sequences without prior signal segmentation.

## When NOT to use

- Your genome is large: Chiron runs at only ~2000 bp/s, making it unsuitable for large genomes
- Your input is already base-called (FASTQ/BAM) rather than raw FAST5 signal — re-base-calling is unnecessary
- You need a production ONT-supported caller with modified-base support rather than an end-to-end research/benchmarking tool

## Inputs

- Raw nanopore FAST5 files containing electrolytic current signal (e.g., example FAST5 reads bundled with the Chiron repository)

## Outputs

- FASTQ file of per-read base-called nucleotide sequences (one entry per input FAST5 read), e.g., chiron_basecalls.fastq

## How to apply

Obtain the raw FAST5 signal reads (e.g., the example FAST5 files bundled with Chiron from github.com/haotianteng/chiron) and point the end-to-end base-caller at the FAST5 directory so the raw ion-current segments are fed directly to the neural network — no segmentation preprocessing, which is the error-prone step relied on by earlier neural-network base-callers such as DeepNano and BasecRAWller. Chiron's CTC-trained model (Connectionist Temporal Classification, in the spirit of Graves et al. 2006 as used in flip-flop model training) decodes each signal segment into a nucleotide sequence in real time at roughly 2000 bp/s. Collect the per-read decoded sequences and write them as FASTQ (one entry per input FAST5 read). As a decision point, be aware Chiron is not suitable for large genomes because of its throughput, so reserve it for small genomes, targeted reads, or benchmarking; alternative HMM/Viterbi-era tools (Metrichor, Nanocall) and preprocessing-dependent callers are not end-to-end.

## Related tools

- **Chiron** (End-to-end deep-learning base-caller: decodes raw FAST5 ion-current segments into nucleotide sequences via a CTC-trained model, outputting FASTQ) — https://github.com/haotianteng/chiron
- **Oxford Nanopore** (Sequencing platform producing the raw FAST5 electrolytic-current signal files that serve as input to the base-caller)
- **Taiyaki** (Related ONT research software for training flip-flop base-calling models used by Guppy; illustrates the CTC-inspired training paradigm underlying end-to-end base-callers) — https://github.com/nanoporetech/taiyaki
- **Bonito** (Open source ONT research basecaller alternative for decoding Oxford Nanopore reads from signal to FASTQ/BAM) — https://github.com/nanoporetech/bonito

## Examples

```
chiron call -i fast5_dir -o chiron_basecalls.fastq --model DNA_default
```

## Evaluation signals

- chiron_basecalls.fastq exists and parses as valid FASTQ
- FASTQ contains exactly one base-called sequence entry per input FAST5 read
- Each FASTQ entry's sequence length is plausible relative to the corresponding read's signal length (given the ~2000 bp/s decoding rate of the model)
- No segmentation preprocessing artifacts appear — the raw signal was fed directly to the neural network as expected for an end-to-end caller
- Output sequences are decodable to standard nucleotide characters (ACGT + ambiguity codes), not alignment scores or signal traces

## Limitations

- Chiron's throughput of ~2000 bp/s makes it unsuitable for base-calling large genomes
- End-to-end deep-learning base-calling is slower than HMM/Viterbi approaches such as Metrichor and Nanocall, which were exclusively adopted for base-calling before deep learning
- Earlier neural-network base-callers (DeepNano, BasecRAWller) required a preprocessing/segmentation step, which end-to-end callers avoid but which may still be encountered in legacy pipelines

## Evidence

- [intro] Chiron is the first end-to-end deep-learning base-caller: [evidence span withheld — anchor status: not_found]
- [readme] Taiyaki trains flip-flop models with CTC-inspired technique: [evidence span withheld — anchor status: external_doc]
- [readme] Bonito basecalls Oxford Nanopore reads from signal: [evidence span withheld — anchor status: external_doc]
