---
name: narrowpeak-coordinate-preprocessing
description: Use when raw narrow-peak BED files (chrom, start, end) from ENCODE/Roadmap
  chromatin profiles must be converted into non-redundant, fixed-width 1000 bp input
  intervals for a Basset/DeepFun CNN training set.
when_to_use_negative:
- Input is already a compendium of fixed-width, non-overlapping intervals (e.g. a
  prepared Basset training compendium) rather than raw narrow-peak calls
- Quantitative signal tracks (fold-enrichment, pileup) are the modeling target — this
  workflow binarizes peaks and discards quantitative epigenomic signal information
- Peak sets from different profiles that must not be collapsed within a profile —
  merging is applied within the same profile only
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: DeepFun
  role: Downstream model whose fixed 1000 bp CNN input window defines the interval
    extension and merging parameters; source of the peak preprocessing workflow
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Base framework providing preprocessing utilities (preprocess_features.py,
    seq_hdf5.py, bedtools dependency) and the 1000 bp fixed input window the intervals
    must match
  repo: https://github.com/davek44/Basset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:04f71f6fe98eb83502f8e0563bff12f51e05385a7d859a5d02cffdc4de848416
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# narrowpeak-coordinate-preprocessing

## Summary

Preprocesses raw ENCODE/Roadmap narrow-peak BED files into fixed 1000 bp genomic training intervals for Basset/DeepFun-style CNNs by extending each peak symmetrically around its midpoint and greedily merging overlaps until no two intervals overlap by more than 200 bp.

## When to use

Use when raw narrow-peak BED files (chrom, start, end) from ENCODE/Roadmap chromatin profiles must be converted into non-redundant, fixed-width 1000 bp input intervals for a Basset/DeepFun CNN training set.

## When NOT to use

- Input is already a compendium of fixed-width, non-overlapping intervals (e.g. a prepared Basset training compendium) rather than raw narrow-peak calls
- Quantitative signal tracks (fold-enrichment, pileup) are the modeling target — this workflow binarizes peaks and discards quantitative epigenomic signal information
- Peak sets from different profiles that must not be collapsed within a profile — merging is applied within the same profile only

## Inputs

- Raw narrow-peak BED files (chrom, start, end coordinates) from ENCODE/Roadmap chromatin profiles (DNase I accessibility, histone marks, transcription factor ChIP-seq)
- bedtools for data preprocessing (per Basset installation requirements)

## Outputs

- merged_peaks.bed: BED-formatted, non-redundant genomic intervals of exactly 1000 bp, no pairwise overlap >200 bp
- Binarized peak labels used as CNN training targets per epigenomic profile

## How to apply

Load the raw narrow-peak BED file for each ENCODE/Roadmap chromatin profile (DNase I accessibility, histone mark, or transcription factor binding) and extend every narrow peak symmetrically ±500 bp around its midpoint to a 1000 bp interval matching the fixed CNN input window of the Basset/DeepFun framework. Sort the extended intervals by chromosome and start coordinate, then iteratively merge overlapping intervals greedily until no two peaks overlap by more than 200 bp, collapsing redundant overlapping calls within the same profile. For each merged cluster, set the merged peak center as the weighted average of the constituent peaks' midpoints. Write the resulting non-redundant peak set as BED-formatted genomic intervals to merged_peaks.bed; validate that every output interval is exactly 1000 bp long and that no two intervals overlap by more than 200 bp. Peaks are subsequently treated as binary labels during model training.

## Related tools

- **DeepFun** (Downstream model whose fixed 1000 bp CNN input window defines the interval extension and merging parameters; source of the peak preprocessing workflow) — https://github.com/bsml320/DeepFun
- **Basset** (Base framework providing preprocessing utilities (preprocess_features.py, seq_hdf5.py, bedtools dependency) and the 1000 bp fixed input window the intervals must match) — https://github.com/davek44/Basset

## Examples

```
python preprocess_features.py --schema SKIP --output merged_peaks.bed encode_roadmap_peaks.bed
```

## Evaluation signals

- Every output interval in merged_peaks.bed is exactly 1000 bp long
- No two intervals in merged_peaks.bed overlap by more than 200 bp
- Each merged peak center equals the weighted average of the constituent merged peaks' midpoints
- All intervals are BED-formatted with valid chrom, start, end coordinates and are sorted by chromosome and start coordinate
- Merged intervals remain within the same profile (no cross-profile collapsing of redundant calls)

## Limitations

- Peaks are binarized for training, so quantitative epigenomic signal information is ignored
- No normalization approach has yet been established to eliminate technical bias for integrating quantitative signals
- The model operates on fixed 1000 bp sequence windows and does not consider long-range interactions such as 3D DNA contact

## Evidence

- [methods] Peak intervals are processed by greedily merging peaks based on distance to adjacent peaks until no peaks overlap by >200 bp, and the merged peak center is set as the weighted average of the merged peaks' midpoints.: [evidence span withheld — anchor status: not_found]
- [discussion] during the training process, we treated the 'peaks' as the binary format to represent the experimentally observed signal around each candidate 'peak' region: [evidence span withheld — anchor status: not_found]
- [readme] You'll also need bedtools for data preprocessing.: [evidence span withheld — anchor status: external_doc]
- [readme] DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python.: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python"
