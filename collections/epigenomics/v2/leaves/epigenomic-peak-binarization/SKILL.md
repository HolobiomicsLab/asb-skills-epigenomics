---
name: epigenomic-peak-binarization
description: Use when downloaded ENCODE/Roadmap narrow-peak BED calls must be converted
  into fixed-length (1000 bp), non-redundant candidate 'peak' regions that serve as
  binary training targets for a Basset/DeepFun CNN.
when_to_use_negative:
- The analysis requires quantitative epigenomic signal intensities — DeepFun binarizes
  peaks and explicitly ignores the quantitative information
- The question depends on long-range interactions such as 3D DNA contact, which the
  resulting sequence-based model does not consider
- Input is already a fixed-length, non-redundant feature compendium (e.g. a Basset-ready
  HDF5 training set) — re-merging would double-process intervals
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Basset
  role: Framework whose preprocessing pipeline (bedtools-based interval handling and
    prepare_compendium tutorial) this peak merging/extension procedure replicates;
    DeepFun is an updated model based on Basset
  repo: https://github.com/davek44/Basset
- name: DeepFun
  role: Downstream CNN whose fixed 1000 bp input window and binary peak labels the
    merged_peaks.bed output is built to serve
  repo: https://github.com/bsml320/DeepFun
- name: bedtools
  role: Used for data preprocessing of BED genomic intervals in the Basset dependency
    chain
  repo: http://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:d6debf63140c59418f20c63d882ae777b5645fe27b95703ea456bf952cdd9ad3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davek44/Basset
  tools:
  - Basset
  - DeepFun
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# epigenomic-peak-binarization

## Summary

Preprocesses raw ENCODE/Roadmap narrow-peak BED intervals into non-redundant 1000 bp genomic training windows for Basset/DeepFun-style CNNs by symmetric midpoint extension, greedy merging (>200 bp overlap tolerance), and weighted-midpoint center placement, so epigenomic profiles can be binarized as training labels.

## When to use

Use when downloaded ENCODE/Roadmap narrow-peak BED calls must be converted into fixed-length (1000 bp), non-redundant candidate 'peak' regions that serve as binary training targets for a Basset/DeepFun CNN.

## When NOT to use

- The analysis requires quantitative epigenomic signal intensities — DeepFun binarizes peaks and explicitly ignores the quantitative information
- The question depends on long-range interactions such as 3D DNA contact, which the resulting sequence-based model does not consider
- Input is already a fixed-length, non-redundant feature compendium (e.g. a Basset-ready HDF5 training set) — re-merging would double-process intervals

## Inputs

- Raw narrow-peak BED files (chrom, start, end) per ENCODE/Roadmap DNase I, histone mark, or transcription factor binding profile
- Reference genome assembly for coordinate-consistent interval handling

## Outputs

- merged_peaks.bed: BED-formatted, non-redundant 1000 bp genomic intervals with merged centers at weighted-average midpoints
- Binarized peak label set per profile for CNN training

## How to apply

Load each raw narrow-peak BED file (chrom, start, end) per ENCODE/Roadmap chromatin profile, then extend every narrow peak symmetrically ±500 bp around its midpoint to a fixed 1000 bp interval matching the Basset CNN input window. Sort the extended intervals by chromosome and start coordinate, then greedily merge clusters iteratively until no two peaks overlap by more than 200 bp, collapsing redundant overlapping calls within the same profile. For each merged cluster, set the merged peak center as the weighted average of the constituent peak midpoints, and write the result to merged_peaks.bed. These preprocessed regions are then used as binary 'peak' labels during training — DeepFun treats the peaks as a binary format representing experimentally observed signal — rather than retaining quantitative epigenomic signal.

## Related tools

- **Basset** (Framework whose preprocessing pipeline (bedtools-based interval handling and prepare_compendium tutorial) this peak merging/extension procedure replicates; DeepFun is an updated model based on Basset) — https://github.com/davek44/Basset
- **DeepFun** (Downstream CNN whose fixed 1000 bp input window and binary peak labels the merged_peaks.bed output is built to serve) — https://github.com/bsml320/DeepFun
- **bedtools** (Used for data preprocessing of BED genomic intervals in the Basset dependency chain) — http://bedtools.readthedocs.io/en/latest/

## Evaluation signals

- Every output interval in merged_peaks.bed is exactly 1000 bp long
- No two intervals in merged_peaks.bed overlap by more than 200 bp
- Each merged peak center equals the weighted average of the constituent peak midpoints
- Output is valid BED format sorted by chromosome and start coordinate
- Each input narrow peak maps to exactly one merged cluster within the same chromatin profile

## Limitations

- Binarizing peaks discards quantitative epigenomic signal information; no normalization approach was established to eliminate technical bias when integrating quantitative signals
- The resulting training scheme does not consider long-range interactions such as 3D DNA contact
- Performance of the downstream CNN may strongly depend on initial convolution filters for captured motif information

## Evidence

- [methods] greedy merge tolerance: [evidence span withheld — anchor status: not_found]
- [discussion] binary peak labels: [evidence span withheld — anchor status: not_found]
- [readme] Basset preprocessing dependency: [evidence span withheld — anchor status: external_doc]
- [readme] compendium preparation tutorial: [evidence span withheld — anchor status: external_doc]
