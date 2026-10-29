---
name: ont-long-read-data-handling
description: Use when raw nanopore FAST5 electrolytic-current signal files must be
  decoded to nucleotide sequences without prior segmentation, e.g. reproducing Chiron-style
  end-to-end deep-learning base-calling.
when_to_use_negative:
- The goal is base-calling large genomes — Chiron runs at only ~2000 bp/s and is explicitly
  noted as not suitable for large genomes; use Guppy or Bonito instead
- Input is already base-called sequence data (FASTQ/BAM) rather than raw FAST5 signal
- A segmentation-based or HMM-based pipeline (e.g. Metrichor, Nanocall) is specifically
  required for comparison — this skill targets end-to-end deep-learning decoding
edam_operation: http://edamontology.org/operation_3458
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Chiron
  role: End-to-end deep-learning base-caller that translates raw FAST5 ion-current
    signal directly to nucleotide sequence via a CTC-trained neural network
  repo: https://github.com/haotianteng/chiron
- name: Oxford Nanopore
  role: Sequencing platform whose devices measure ion flow through a nanopore, producing
    the raw signal decoded by the base-caller
- name: Bonito
  role: Open-source ONT research basecaller, the successor resource largely replacing
    Taiyaki; suitable when Chiron throughput is insufficient
  repo: https://github.com/nanoporetech/bonito
- name: Taiyaki
  role: Research software for training neural-network basecalling models (flip-flop,
    CTC-inspired) used in Guppy and megalodon; relevant for training custom base-callers
  repo: https://github.com/nanoporetech/taiyaki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xinn.2021.100153
    title: Applications and potentials of nanopore sequencing in the (epi)genome and
      (epi)transcriptome era
  source_content_hash: sha256:f1d3a3037d46c21dd5e5af02cd8315472413d72f3f3071c69a9c38ba21a8793f
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
  edam_operation: http://edamontology.org/operation_3458
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# ont-long-read-data-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

End-to-end deep-learning base-calling of Oxford Nanopore raw FAST5 ion-current signal into nucleotide sequence using Chiron, the first deep-learning model to achieve end-to-end base-calling without error-prone signal segmentation, with FASTQ output validation.

## When to use

Use when raw nanopore FAST5 electrolytic-current signal files must be decoded to nucleotide sequences without prior segmentation, e.g. reproducing Chiron-style end-to-end deep-learning base-calling.

## When NOT to use

- The goal is base-calling large genomes — Chiron runs at only ~2000 bp/s and is explicitly noted as not suitable for large genomes; use Guppy or Bonito instead
- Input is already base-called sequence data (FASTQ/BAM) rather than raw FAST5 signal
- A segmentation-based or HMM-based pipeline (e.g. Metrichor, Nanocall) is specifically required for comparison — this skill targets end-to-end deep-learning decoding

## Inputs

- Raw Oxford Nanopore FAST5 files containing electrolytic ion-current signal per read
- Example FAST5 reads bundled with the Chiron repository (github.com/haotianteng/chiron)

## Outputs

- FASTQ file (e.g. chiron_basecalls.fastq) with one base-called nucleotide sequence entry per input FAST5 read

## How to apply

Obtain raw-signal FAST5 reads (e.g. the example FAST5 files bundled with Chiron from github.com/haotianteng/chiron). Launch Chiron pointing at the FAST5 directory so the raw ion-current segments are fed directly to the CTC-trained neural network, skipping any prior segmentation of the electrolytic signal — this is the defining advantage over early neural-network base-callers like DeepNano and BasecRAWller, which relied on a preprocessing/segmentation step. Chiron's CTC-trained model decodes each signal segment into a nucleotide sequence at roughly 2000 bp/s. Collect the per-read decoded sequences and write them in FASTQ format. Validate that the output FASTQ exists, parses as valid FASTQ, and contains one base-called sequence entry per input FAST5 read. Note that Chiron's throughput makes it unsuitable for large genomes, where callers like Guppy (models trained with Taiyaki's flip-flop/CTC-inspired technique) or Bonito are preferred.

## Related tools

- **Chiron** (End-to-end deep-learning base-caller that translates raw FAST5 ion-current signal directly to nucleotide sequence via a CTC-trained neural network) — https://github.com/haotianteng/chiron
- **Oxford Nanopore** (Sequencing platform whose devices measure ion flow through a nanopore, producing the raw signal decoded by the base-caller)
- **Bonito** (Open-source ONT research basecaller, the successor resource largely replacing Taiyaki; suitable when Chiron throughput is insufficient) — https://github.com/nanoporetech/bonito
- **Taiyaki** (Research software for training neural-network basecalling models (flip-flop, CTC-inspired) used in Guppy and megalodon; relevant for training custom base-callers) — https://github.com/nanoporetech/taiyaki

## Examples

```
chiron call -i example_fast5_dir -o chiron_basecalls.fastq
```

## Evaluation signals

- chiron_basecalls.fastq exists and parses as valid FASTQ
- Output FASTQ contains exactly one base-called sequence entry per input FAST5 read
- No separate segmentation/preprocessing step was required between FAST5 input and network decoding (end-to-end property)
- Decoded base-called sequences align to expected reference (e.g. phi X174 or sample-of-origin) with plausible per-read accuracy
- Base-calling throughput is in the range reported for Chiron (~2000 bp/s)

## Limitations

- Chiron runs at ~2000 bp/s and is not suitable for large genomes
- Nanopore raw signal is complex with long-range dependencies; model quality depends on training regime (e.g. flip-flop models trained with Taiyaki, CTC-inspired)
- For anything other than basecaller training or method development, ONT recommends dorado over research basecallers like Bonito

## Evidence

- [intro] First deep-learning end-to-end base-caller: [evidence span withheld — anchor status: not_found]
- [readme] Taiyaki purpose: [evidence span withheld — anchor status: external_doc]
- [readme] Bonito usage and recommendation: [evidence span withheld — anchor status: external_doc]
