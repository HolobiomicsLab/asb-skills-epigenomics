---
name: weighted-midpoint-center-computation
description: Use when raw narrow-peak BED files from ENCODE/Roadmap chromatin profiles
  must be converted into fixed-length (1000 bp), non-overlapping (>200 bp) genomic
  training intervals for a Basset/DeepFun CNN input.
when_to_use_negative:
- Quantitative epigenomic signal information is required — this skill treats 'peaks'
  in binary format and ignores quantitative signal, which DeepFun's authors explicitly
  note as a limitation
- Long-range interactions such as 3D DNA contact are part of the modeling goal — the
  DeepFun/Basset CNN model does not consider them
- The input peak set is already a preprocessed, non-redundant 1000 bp interval compendium
  (e.g. an existing Basset compendium)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: DeepFun
  role: Downstream CNN whose fixed 1000 bp input window and binary peak training format
    dictate the extension threshold and merge criterion
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Underlying framework providing the preprocessing pipeline (preprocess_features.py,
    seq_hdf5.py) and bedtools-based data preprocessing conventions
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:8b842035fef21e792dc0edecb1c129746bd530efb4db90490fa3e3918216d26b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Weighted midpoint peak center computation

## Summary

Preprocesses raw ENCODE/Roadmap narrow-peak BED intervals into non-redundant 1000 bp training intervals for Basset/DeepFun-style CNNs by extending each peak ±500 bp around its midpoint, greedily merging overlapping intervals (>200 bp overlap), and setting each merged peak center to the weighted average of constituent peak midpoints.

## When to use

Use when raw narrow-peak BED files from ENCODE/Roadmap chromatin profiles must be converted into fixed-length (1000 bp), non-overlapping (>200 bp) genomic training intervals for a Basset/DeepFun CNN input.

## When NOT to use

- Quantitative epigenomic signal information is required — this skill treats 'peaks' in binary format and ignores quantitative signal, which DeepFun's authors explicitly note as a limitation
- Long-range interactions such as 3D DNA contact are part of the modeling goal — the DeepFun/Basset CNN model does not consider them
- The input peak set is already a preprocessed, non-redundant 1000 bp interval compendium (e.g. an existing Basset compendium)

## Inputs

- Raw narrow-peak BED files (chrom, start, end coordinates) from ENCODE/Roadmap DNase I accessibility, histone mark, and transcription factor binding (ChIP-seq) profiles

## Outputs

- merged_peaks.bed — BED-formatted, non-redundant 1000 bp genomic training intervals with weighted-average peak centers

## How to apply

Load the raw narrow-peak BED file (chrom, start, end) for each ENCODE/Roadmap chromatin profile, then extend every narrow peak symmetrically ±500 bp around its midpoint to yield exactly 1000 bp intervals matching the fixed CNN input window of the Basset/DeepFun framework. Sort the extended intervals by chromosome and start coordinate to prepare for greedy merging, then iteratively merge overlapping intervals greedily until no two peaks overlap by more than 200 bp, collapsing redundant overlapping calls within the same profile. For each merged cluster, set the merged peak center as the weighted average of the constituent peaks' midpoints rather than a simple cluster midpoint, preserving the contribution of each original peak. Write the resulting non-redundant peak set to merged_peaks.bed in BED format; peaks are used in binary format during training to represent experimentally observed signal around each candidate peak region.

## Related tools

- **DeepFun** (Downstream CNN whose fixed 1000 bp input window and binary peak training format dictate the extension threshold and merge criterion) — https://github.com/bsml320/DeepFun
- **Basset** (Underlying framework providing the preprocessing pipeline (preprocess_features.py, seq_hdf5.py) and bedtools-based data preprocessing conventions) — https://github.com/davek44/Basset

## Examples

```
bedtools sort -i peaks_1000bp.bed | bedtools merge -d 200 > merged_peaks.bed
```

## Evaluation signals

- Every output interval in merged_peaks.bed is exactly 1000 bp long (each raw peak was extended ±500 bp around its midpoint)
- No two intervals in merged_peaks.bed overlap by more than 200 bp after greedy merging
- Each merged peak center equals the weighted average of the constituent peaks' midpoints
- Output file is valid BED format (chrom, start, end) and sorted by chromosome and start coordinate
- Number of output intervals is less than or equal to the number of input narrow peaks (redundant overlapping calls collapsed)

## Limitations

- Peaks are binarized, so quantitative epigenomic signal information is ignored during training
- No normalization approach has been established to eliminate technical bias when integrating quantitative signals across profiles
- The Basset/DeepFun CNN window (1000 bp) cannot represent long-range interactions such as 3D DNA contact
- Merging is per-profile; cross-profile redundancy is not collapsed by this procedure

## Evidence

- [discussion] Peaks are treated as binary training labels: [evidence span withheld — anchor status: not_found]
- [methods] DeepFun is built on the Basset framework: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python."
- [readme] bedtools is required for data preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] Basset compendium preprocessing tutorials exist: [evidence span withheld — anchor status: external_doc]
