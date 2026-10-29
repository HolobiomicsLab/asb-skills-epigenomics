---
name: deep-learning-ctc-sequence-decoding
description: Use when raw Oxford Nanopore FAST5 signal files must be base-called directly
  without an event-detection/segmentation preprocessing step, or when comparing end-to-end
  deep-learning base-callers.
when_to_use_negative:
- Base-calling large genomes — Chiron runs at ~2000 bp/s and is explicitly noted as
  not suitable for large genomes
- The input is already base-called (FASTQ/BAM); this skill operates on raw FAST5 signal
  only
- A segmented/event-detected signal is required by the downstream tool — end-to-end
  models bypass segmentation entirely
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: First end-to-end deep-learning base-caller; CTC-trained model decodes raw
    FAST5 signal segments directly to nucleotide sequences
  repo: https://github.com/haotianteng/chiron
- name: Oxford Nanopore
  role: Sequencing platform producing the raw ion-current FAST5 signal that the network
    decodes
- name: Taiyaki
  role: ONT research software for training basecaller models (e.g. flip-flop models,
    a CTC-inspired technique) used in Guppy and megalodon
  repo: https://github.com/nanoporetech/taiyaki
- name: Bonito
  role: ONT open-source research basecaller; successor resource largely replacing
    Taiyaki for basecaller training and method development
  repo: https://github.com/nanoporetech/bonito
- name: Causalcall
  role: End-to-end temporal convolution-based deep-learning base-caller alternative
  repo: https://github.com/scutbioinformatic/causalcall
- name: URnano
  role: Refined U-net (UR-net) model improving previous end-to-end deep-learning base-callers
  repo: https://github.com/yaozhong/URnano
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:1d9a8e5629a4152648ab046479108d5ceb62f19213c4dd772bd23b70c7b69703
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
  - Oxford Nanopore
  - Taiyaki
  - URnano
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# deep-learning-ctc-sequence-decoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end deep-learning decoding of raw nanopore ion-current signal (FAST5) into nucleotide sequences using CTC-trained neural networks such as Chiron, avoiding error-prone segmentation of the signal into individual base events.

## When to use

Use when raw Oxford Nanopore FAST5 signal files must be base-called directly without an event-detection/segmentation preprocessing step, or when comparing end-to-end deep-learning base-callers.

## When NOT to use

- Base-calling large genomes — Chiron runs at ~2000 bp/s and is explicitly noted as not suitable for large genomes
- The input is already base-called (FASTQ/BAM); this skill operates on raw FAST5 signal only
- A segmented/event-detected signal is required by the downstream tool — end-to-end models bypass segmentation entirely

## Inputs

- Raw Oxford Nanopore FAST5 signal files (raw ion-current segments)
- Example FAST5 reads bundled with the Chiron repository

## Outputs

- Per-read base-called nucleotide sequences in FASTQ format (chiron_basecalls.fastq)

## How to apply

Obtain raw-signal FAST5 reads (e.g. the example reads bundled with Chiron from github.com/haotianteng/chiron) and feed the raw ion-current segments directly to the neural network — no prior segmentation step is performed, which is what distinguishes end-to-end models from early neural-network base-callers like DeepNano and BasecRAWller. The network is trained with Connectionist Temporal Classification (CTC) loss so it can decode variable-length nucleotide sequences from the continuous signal without per-base alignment labels. Collect the per-read decoded sequences and write them in FASTQ format (e.g. chiron_basecalls.fastq). Note the throughput trade-off: Chiron runs at roughly 2000 bp/s, which is adequate for small targets but not for large genomes — for higher-throughput or production use, ONT's Guppy (trained via Taiyaki with flip-flop models, a CTC-inspired technique) or Bonito are the supported alternatives.

## Related tools

- **Chiron** (First end-to-end deep-learning base-caller; CTC-trained model decodes raw FAST5 signal segments directly to nucleotide sequences) — https://github.com/haotianteng/chiron
- **Oxford Nanopore** (Sequencing platform producing the raw ion-current FAST5 signal that the network decodes)
- **Taiyaki** (ONT research software for training basecaller models (e.g. flip-flop models, a CTC-inspired technique) used in Guppy and megalodon) — https://github.com/nanoporetech/taiyaki
- **Bonito** (ONT open-source research basecaller; successor resource largely replacing Taiyaki for basecaller training and method development) — https://github.com/nanoporetech/bonito
- **Causalcall** (End-to-end temporal convolution-based deep-learning base-caller alternative) — https://github.com/scutbioinformatic/causalcall
- **URnano** (Refined U-net (UR-net) model improving previous end-to-end deep-learning base-callers) — https://github.com/yaozhong/URnano

## Examples

```
chiron call -i example_fast5_dir/ -o chiron_basecalls.fastq
```

## Evaluation signals

- chiron_basecalls.fastq exists and parses as valid FASTQ
- One base-called sequence entry is present per input FAST5 read
- No prior event-detection/segmentation step was run — raw signal segments went directly into the network
- Per-read read lengths are plausible relative to expected template lengths for the sequenced sample
- If compared against alternatives (Causalcall, URnano, Guppy), read accuracy is assessed on a common reference

## Limitations

- Chiron runs at ~2000 bp/s and is not suitable for large genomes
- Early neural-network base-callers (DeepNano, BasecRAWller) required a preprocessing/segmentation step, so they are not end-to-end and should not be treated as equivalent
- Taiyaki is now unsupported; ONT recommends Bonito (or dorado for non-training use) instead

## Evidence

- [intro] Chiron is the first deep-learning model to achieve end-to-end base-calling, avoiding error-prone segmentation of ion current signals; it runs at ~2000 bp/s but is not suitable for large genomes.: [evidence span withheld — anchor status: not_found]
- [intro] Early neural-network base-callers relied on a preprocessing (segmentation) step, unlike end-to-end models.: "Early neural-network base-callers (such as DeepNano41 and BasecRAWller42) relied on a preprocessing step"
- [readme] Taiyaki trains CTC-inspired flip-flop models used by Guppy and megalodon.: "This includes the flip-flop models, which are trained using a technique inspired by Connectionist Temporal Classification
(Graves et al 2006)."
- [readme] Taiyaki is unsupported and replaced by Bonito.: [evidence span withheld — anchor status: external_doc]
