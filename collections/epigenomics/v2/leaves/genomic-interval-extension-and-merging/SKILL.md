---
name: genomic-interval-extension-and-merging
description: Use when raw ChIP-seq/DNase narrow-peak BED files (chrom, start, end)
  must be converted into fixed-width genomic training windows with a fixed CNN input
  size and no redundant overlap before training.
when_to_use_negative:
- Input is already a fixed-width, non-redundant interval set (e.g. a prior merged_peaks.bed)
  — re-merging would alter validated training regions
- The task is variant annotation or SNP Activity Difference computation with a pre-trained
  model (dnacnn_best_A/B.zip) rather than building the peak label compendium from
  scratch
- Quantitative signal (peak score) information is required, since this skill binarizes
  peaks and discards signal intensity
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Basset
  role: Framework whose preprocessing pipeline (bedtools, preprocess_features.py,
    seq_hdf5.py) implements interval extension and merging before CNN training; DeepFun
    is an updated model based on it
  repo: https://github.com/davek44/Basset
- name: DeepFun
  role: 'Downstream consumer: the merged 1000 bp peak intervals become the binary
    training labels for the 7879-profile CNN'
  repo: https://github.com/bsml320/DeepFun
- name: bedtools
  role: Data preprocessing dependency for interval operations in the Basset pipeline
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:a7ab24a40c6a2b5c1e3ffd09e22a6e52fd0ff5b97b1514924b96e9461d2add86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davek44/Basset
  tools:
  - Basset
  - DeepFun
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# genomic-interval-extension-and-merging

## Summary

Preprocessing of ENCODE/Roadmap narrow-peak BED intervals into fixed-width, non-redundant 1000 bp training regions for Basset/DeepFun CNN models: symmetrical extension around peak midpoints followed by greedy merging of overlapping intervals.

## When to use

Use when raw ChIP-seq/DNase narrow-peak BED files (chrom, start, end) must be converted into fixed-width genomic training windows with a fixed CNN input size and no redundant overlap before training.

## When NOT to use

- Input is already a fixed-width, non-redundant interval set (e.g. a prior merged_peaks.bed) — re-merging would alter validated training regions
- The task is variant annotation or SNP Activity Difference computation with a pre-trained model (dnacnn_best_A/B.zip) rather than building the peak label compendium from scratch
- Quantitative signal (peak score) information is required, since this skill binarizes peaks and discards signal intensity

## Inputs

- Raw narrow-peak BED files (chrom, start, end) from ENCODE/Roadmap chromatin profiles (DNase accessibility, histone marks, transcription factor ChIP-seq)
- Chromosome sizes / reference genome assembly for midpoint computation and sorting

## Outputs

- merged_peaks.bed: BED-formatted set of exactly 1000 bp intervals with no pair overlapping by more than 200 bp
- One non-redundant peak set per ENCODE/Roadmap profile for downstream seq_hdf5.py / CNN training input

## How to apply

Load each raw narrow-peak BED file (chrom, start, end) per ENCODE/Roadmap chromatin profile and extend every narrow peak symmetrically to 1000 bp (±500 bp around its midpoint) to match the fixed CNN input window of the Basset/DeepFun framework. Sort the extended intervals by chromosome and start coordinate, then iteratively merge them greedily until no two intervals overlap by more than 200 bp, collapsing redundant calls within the same profile. For each merged cluster, define the merged peak center as the weighted average of the constituent peaks' midpoints, and write the non-redundant interval set to merged_peaks.bed in BED format. The rationale is that DeepFun treats peaks as binary labels over candidate regions, so each profile needs exactly one 1000 bp window per functional site; Basset's preprocessing stack (bedtools plus preprocess_features.py / seq_hdf5.py) is the reference implementation for this stage.

## Related tools

- **Basset** (Framework whose preprocessing pipeline (bedtools, preprocess_features.py, seq_hdf5.py) implements interval extension and merging before CNN training; DeepFun is an updated model based on it) — https://github.com/davek44/Basset
- **DeepFun** (Downstream consumer: the merged 1000 bp peak intervals become the binary training labels for the 7879-profile CNN) — https://github.com/bsml320/DeepFun
- **bedtools** (Data preprocessing dependency for interval operations in the Basset pipeline) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
export BASSETDIR=the/dir/where/basset/is/installed; python $BASSETDIR/preprocess_features.py
```

## Evaluation signals

- Every output interval in merged_peaks.bed is exactly 1000 bp long (±500 bp around the original peak midpoint)
- No two intervals within a profile's merged_peaks.bed overlap by more than 200 bp
- Each merged cluster's center equals the weighted average of the constituent peaks' midpoints
- All intervals are sorted by chromosome and start coordinate before merging completes
- One peak set is produced per ENCODE/Roadmap profile so the feature space of 1548 DNase I, 1536 histone mark and 4795 TF binding profiles can be binarized consistently

## Limitations

- Binarizing peaks discards quantitative epigenomic signal information, which the authors acknowledge as ignored ('However, it has ignored the quantitative information.')
- The 1000 bp window captures only local sequence context; DeepFun does not model long-range interactions such as 3D DNA contact
- The Basset preprocessing stack requires Torch7/Python and bedtools dependencies installed per Basset tutorials, and depends on the BASSETDIR environment variable

## Evidence

- [methods] Peak intervals are processed by greedily merging peaks based on distance to adjacent peaks until no peaks overlap by >200 bp: [evidence span withheld — anchor status: not_found]
- [methods] Extend every narrow peak symmetrically around its midpoint to a 1000 bp interval (±500 bp from the midpoint) to match the fixed CNN input window: [evidence span withheld — anchor status: not_found]
- [readme] You'll also need [bedtools](http://bedtools.readthedocs.io/en/latest/) for data preprocessing.: [evidence span withheld — anchor status: external_doc]
- [discussion] during the training process, we treated the 'peaks' as the binary format to represent the experimentally observed signal around each candidate 'peak' region: [evidence span withheld — anchor status: not_found]
- [readme] DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python.: "DeepFun is an updated model based on Basset Framework (https://github.com/davek44/Basset) based on Torch7 and Python"
