---
name: fast5-electrolytic-current-decoding
description: Use when starting from Oxford Nanopore raw FAST5 electrolytic-current
  signal files and you need to decode them into per-read nucleotide sequences (FASTQ)
  rather than working from pre-base-called reads.
when_to_use_negative:
- Input is already base-called FASTQ/BAM sequence data — decoding FAST5 signal would
  be redundant
- The task requires base-calling very large genomes — Chiron runs at ~2000 bp/s and
  is not suitable for large genomes
- The workflow requires modified-base calling with ONT-supported mods — the Bonito
  README directs users to dorado instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: First end-to-end deep-learning base-caller; decodes raw FAST5 ion-current
    segments directly to nucleotide sequences via a CTC-trained model
  repo: https://github.com/haotianteng/chiron
- name: Causalcall
  role: End-to-end temporal convolution-based deep-learning base-calling alternative
  repo: https://github.com/scutbioinformatic/causalcall
- name: URnano
  role: Refined U-net (UR-net) model improving previous end-to-end deep-learning base-callers
  repo: https://github.com/yaozhong/URnano
- name: Taiyaki
  role: Research software for training base-calling models (flip-flop, CTC-inspired)
    for Oxford Nanopore reads; exportable to Guppy and megalodon
  repo: https://github.com/nanoporetech/taiyaki
- name: Bonito
  role: Open source research basecaller for Oxford Nanopore reads; use for basecaller
    training or method development
  repo: https://github.com/nanoporetech/bonito
- name: Nanocall
  role: Earlier HMM/Viterbi-decoding base-caller for nanopore signal (pre-deep-learning
    generation)
  repo: https://github.com/mateidavid/nanocall
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:757b39a9add674f97148be0c6dd6805a2afd900219c14d9aa02e83c4938de1f0
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
  - Causalcall
  - Chiron
  - Nanocall
  - Taiyaki
  - URnano
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fast5-electrolytic-current-decoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Base-calling of Oxford Nanopore reads: translating raw electrolytic ion-current signal stored in FAST5 files into nucleotide sequences, using end-to-end deep-learning models such as Chiron's CTC-trained network that bypass error-prone signal segmentation.

## When to use

Use when starting from Oxford Nanopore raw FAST5 electrolytic-current signal files and you need to decode them into per-read nucleotide sequences (FASTQ) rather than working from pre-base-called reads.

## When NOT to use

- Input is already base-called FASTQ/BAM sequence data — decoding FAST5 signal would be redundant
- The task requires base-calling very large genomes — Chiron runs at ~2000 bp/s and is not suitable for large genomes
- The workflow requires modified-base calling with ONT-supported mods — the Bonito README directs users to dorado instead

## Inputs

- Raw Oxford Nanopore FAST5 files containing electrolytic ion-current signal per read
- A pretrained end-to-end deep-learning base-calling model (e.g., Chiron's CTC-trained model)

## Outputs

- Per-read base-called nucleotide sequences in FASTQ format (e.g., chiron_basecalls.fastq)

## How to apply

Obtain the raw-signal FAST5 reads (e.g., the example reads bundled with Chiron from github.com/haotianteng/chiron). Launch an end-to-end deep-learning base-caller such as Chiron, which feeds raw ion-current segments directly to a neural network with no prior segmentation step — unlike early neural-network base-callers (DeepNano, BasecRAWller) that relied on preprocessing. The CTC-trained model decodes each signal segment into a nucleotide sequence. Collect the per-read decoded sequences and write them in FASTQ format (e.g., chiron_basecalls.fastq). Rationale: Chiron was the first deep-learning model to achieve end-to-end base-calling, running at ~2000 bp/s; alternative HMM/Viterbi approaches (Metrichor, Nanocall) were the earlier generation of base-callers, and newer models (Causalcall, URnano) refine the end-to-end deep-learning strategy. If training or model development rather than base-calling is the goal, ONT's Taiyaki/Bonito stack trains flip-flop models exported for use in Guppy.

## Related tools

- **Chiron** (First end-to-end deep-learning base-caller; decodes raw FAST5 ion-current segments directly to nucleotide sequences via a CTC-trained model) — https://github.com/haotianteng/chiron
- **Causalcall** (End-to-end temporal convolution-based deep-learning base-calling alternative) — https://github.com/scutbioinformatic/causalcall
- **URnano** (Refined U-net (UR-net) model improving previous end-to-end deep-learning base-callers) — https://github.com/yaozhong/URnano
- **Taiyaki** (Research software for training base-calling models (flip-flop, CTC-inspired) for Oxford Nanopore reads; exportable to Guppy and megalodon) — https://github.com/nanoporetech/taiyaki
- **Bonito** (Open source research basecaller for Oxford Nanopore reads; use for basecaller training or method development) — https://github.com/nanoporetech/bonito
- **Nanocall** (Earlier HMM/Viterbi-decoding base-caller for nanopore signal (pre-deep-learning generation)) — https://github.com/mateidavid/nanocall

## Examples

```
chiron call -i example_fast5_dir/ -o chiron_basecalls.fastq
```

## Evaluation signals

- chiron_basecalls.fastq exists and parses as valid FASTQ
- The FASTQ contains one base-called sequence entry per input FAST5 read
- Base-called reads can be aligned against the reference (e.g., phi X174) with expected identity, distinguishing end-to-end decoding from the error-prone segmentation approach
- Throughput is in the reported range (~2000 bp/s for Chiron), indicating the CTC model is decoding rather than stalling on segmentation
- For Taiyaki-trained models, the workflow Makefile completes from fast5 files through basecalling and exported models run in Guppy

## Limitations

- Chiron runs at ~2000 bp/s and is not suitable for large genomes
- Early neural-network base-callers (DeepNano, BasecRAWller) require a preprocessing/segmentation step that is error-prone
- Taiyaki repository is now unsupported; ONT recommends Bonito (and for production use, dorado)
- For modified-base calling with ONT-supported mods, Bonito README directs users to dorado and remora instead

## Evidence

- [intro] Chiron is the first deep-learning model to achieve end-to-end base-calling, avoiding error-prone segmentation of ion current signals: [evidence span withheld — anchor status: not_found]
- [intro] HMM/Viterbi base-callers preceded deep learning: "Hidden Markov Model (HMM) and Viterbi decoding approaches, such as Metrichor and Nanocall, were exclusively adopted for base-calling"
- [intro] Early neural-network base-callers needed preprocessing: "Early neural-network base-callers (such as DeepNano41 and BasecRAWller42) relied on a preprocessing step"
- [intro] FAST5 signals decode to sequence and current information: "Raw electrolytic current signal files (FAST5) can be decoded to sequence information and electrolytic current signal information"
- [readme] Taiyaki purpose and CTC-inspired flip-flop training: [evidence span withheld — anchor status: external_doc]
- [readme] Bonito is a research basecaller with a caveat: [evidence span withheld — anchor status: external_doc]
