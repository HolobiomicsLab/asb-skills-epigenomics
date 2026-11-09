---
name: fastq-read-output-generation
description: Use when the input is raw nanopore FAST5 signal reads and the goal is
  base-calling each read into a nucleotide sequence collected in a single FASTQ file,
  without prior signal segmentation.
when_to_use_negative:
- Input is already base-called sequence data (e.g., existing FASTQ/BAM basecalls)
  rather than raw FAST5 signal files
- The target genome is large — Chiron runs at ~2000 bp/s and is explicitly noted as
  not suitable for large genomes
- A modified-base detection model is required, which ONT directs to dorado/megalodon
  rather than this base-calling workflow
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Chiron
  role: End-to-end deep-learning base-caller that decodes raw FAST5 ion-current segments
    directly to nucleotide sequences via CTC, writing per-read FASTQ output
  repo: https://github.com/haotianteng/chiron
- name: Oxford Nanopore
  role: Sequencing platform whose devices measure ion flow through a nanopore, producing
    the raw FAST5 signals that this base-calling step converts to FASTQ
- name: Bonito
  role: Alternative open-source ONT research basecaller that also supports writing
    FASTQ output from raw reads
  repo: https://github.com/nanoporetech/bonito
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:133fecae8652cfb349dade64063196b4c28ccf0d46823255f30a552c2de70753
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fastq-read-output-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a per-read FASTQ output file by decoding raw Oxford Nanopore FAST5 ion-current signals with an end-to-end deep-learning base-caller (Chiron), writing one base-called nucleotide sequence per input read. This step converts the raw signal layer of nanopore sequencing into sequence information usable by alignment, assembly, and variant-calling steps.

## When to use

Use when the input is raw nanopore FAST5 signal reads and the goal is base-calling each read into a nucleotide sequence collected in a single FASTQ file, without prior signal segmentation.

## When NOT to use

- Input is already base-called sequence data (e.g., existing FASTQ/BAM basecalls) rather than raw FAST5 signal files
- The target genome is large — Chiron runs at ~2000 bp/s and is explicitly noted as not suitable for large genomes
- A modified-base detection model is required, which ONT directs to dorado/megalodon rather than this base-calling workflow

## Inputs

- Raw FAST5 files containing Oxford Nanopore electrolytic current signals (one per read)
- Chiron deep-learning base-calling model (CTC-trained, from github.com/haotianteng/chiron)

## Outputs

- chiron_basecalls.fastq — one base-called nucleotide sequence entry per input FAST5 read, in FASTQ format

## How to apply

Obtain the example FAST5 raw-signal reads (e.g., from the Chiron repository github.com/haotianteng/chiron). Point Chiron — the first deep-learning model to achieve end-to-end base-calling — at the FAST5 directory so raw ion-current segments are fed directly to the CTC-trained neural network with no error-prone segmentation preprocessing (a contrast to early neural-network base-callers like DeepNano and BasecRAWller, which required a preprocessing step). Let the CTC model decode each signal segment into a nucleotide sequence at roughly 2000 bp/s, then collect the per-read decoded sequences and write them in FASTQ format to chiron_basecalls.fastq. The base-calling step is foundational: the resulting FASTQ feeds downstream long-read alignment, assembly, and variant-calling stages reviewed in the article.

## Related tools

- **Chiron** (End-to-end deep-learning base-caller that decodes raw FAST5 ion-current segments directly to nucleotide sequences via CTC, writing per-read FASTQ output) — https://github.com/haotianteng/chiron
- **Oxford Nanopore** (Sequencing platform whose devices measure ion flow through a nanopore, producing the raw FAST5 signals that this base-calling step converts to FASTQ)
- **Bonito** (Alternative open-source ONT research basecaller that also supports writing FASTQ output from raw reads) — https://github.com/nanoporetech/bonito

## Examples

```
chiron call -i example_fast5_dir -o chiron_basecalls.fastq
```

## Evaluation signals

- chiron_basecalls.fastq exists and parses as valid FASTQ (4-line records with @/+, matching header IDs and sequence/quality lengths)
- The FASTQ contains exactly one base-called sequence entry per input FAST5 read
- No prior segmentation step was applied — the workflow fed raw ion-current segments directly to the neural network (end-to-end property)
- Per-read decoded sequences are non-empty nucleotide strings (ACGTN) of plausible length for the input signal segments
- Downstream consumers (alignment/assembly steps reviewed in the article) can read the FASTQ without format errors

## Limitations

- Chiron's throughput (~2000 bp/s) makes it unsuitable for large genomes
- Chiron was reported before newer ONT basecallers; Bonito's README points to dorado for anything other than basecaller training or method development, and taiyaki is deprecated/unsupported
- Base-calling quality limits downstream steps: the article notes InDel detection sensitivity around 60% with accuracy around 90%, so base-call errors propagate to variant calling

## Evidence

- [intro] Teng et al. reported an algorithm called Chiron, which was the first deep-learning model to achieve end-to-end base-calling: [evidence span withheld — anchor status: not_found]
- [other] Chiron is the first deep-learning model to achieve end-to-end base-calling, avoiding error-prone segmentation of ion current signals; it runs at ~2000 bp/s: [evidence span withheld — anchor status: not_found]
- [readme] Bonito supports writing aligned/unaligned `{fastq, sam, bam, cram}`.: [evidence span withheld — anchor status: external_doc]
