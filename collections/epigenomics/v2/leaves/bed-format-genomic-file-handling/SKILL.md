---
name: bed-format-genomic-file-handling
description: Use when raw narrow-peak BED files (chrom, start, end) from ENCODE/Roadmap
  chromatin profiles must become fixed 1000 bp, non-redundant training intervals for
  a Basset/DeepFun CNN input window.
when_to_use_negative:
- The peak set is already a non-redundant compendium of fixed-length intervals (e.g.
  a pre-built Basset compendium), so re-extending and re-merging would distort centers
- The analysis requires quantitative epigenomic signal per region — DeepFun binarizes
  peaks and explicitly ignores quantitative information
- The task is variant-effect prediction from a VCF using the pre-trained model, which
  consumes SNPs rather than re-preprocessed peak BEDs
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: DeepFun
  role: Consumes the preprocessed 1000 bp merged peak intervals as binary training
    labels for its dense-epigenomic CNN
  repo: https://github.com/bsml320/DeepFun
- name: Basset
  role: Defines the 1000 bp CNN input window and the preprocessing pipeline (bedtools
    dependency) that DeepFun is built on
  repo: https://github.com/davek44/Basset
- name: bedtools
  role: Named dependency in the Basset README for BED interval data preprocessing
    (extension/sorting/merging of peak intervals)
  repo: http://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab429
    title: 'DeepFun: a deep learning sequence-based model to decipher non-coding variant
      effect in a tissue- and cell type-specific manner'
  source_content_hash: sha256:d04fd1b21ed475518a44e08025f0ae39487d037ebfd143da9b37463c2c4945e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/DeepFun
  tools:
  - Basset
  - DeepFun
  - bedtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# bed-format-genomic-file-handling

## Summary

Preprocess raw ENCODE/Roadmap narrow-peak BED intervals into fixed-length, non-redundant genomic training windows (1000 bp extension, greedy merge with a 200 bp overlap cutoff, weighted-midpoint centers) for Basset/DeepFun CNN training.

## When to use

Use when raw narrow-peak BED files (chrom, start, end) from ENCODE/Roadmap chromatin profiles must become fixed 1000 bp, non-redundant training intervals for a Basset/DeepFun CNN input window.

## When NOT to use

- The peak set is already a non-redundant compendium of fixed-length intervals (e.g. a pre-built Basset compendium), so re-extending and re-merging would distort centers
- The analysis requires quantitative epigenomic signal per region — DeepFun binarizes peaks and explicitly ignores quantitative information
- The task is variant-effect prediction from a VCF using the pre-trained model, which consumes SNPs rather than re-preprocessed peak BEDs

## Inputs

- Raw narrow-peak BED file (chrom, start, end) per ENCODE/Roadmap chromatin profile (DNase I accessibility, histone mark, or transcription factor binding)
- Peak coordinates sorted by chromosome and start coordinate after 1000 bp extension

## Outputs

- merged_peaks.bed: BED-formatted, non-redundant 1000 bp genomic training intervals with weighted-average peak centers
- Binary peak labels per candidate region for Basset/DeepFun CNN training

## How to apply

Load the raw narrow-peak BED file (chrom, start, end) for each ENCODE/Roadmap profile and extend every narrow peak symmetrically around its midpoint to a 1000 bp interval (±500 bp), matching the fixed CNN input window of the Basset/DeepFun framework. Sort the extended intervals by chromosome and start coordinate, then iteratively merge overlapping intervals greedily until no two peaks overlap by more than 200 bp, collapsing redundant calls within the same profile. For each merged cluster, set the merged peak center as the weighted average of the constituent peak midpoints. Write the result as BED-formatted genomic intervals (merged_peaks.bed); bedtools is the documented preprocessing dependency in the Basset README, and the intervals are then used as binary peak labels during CNN training.

## Related tools

- **DeepFun** (Consumes the preprocessed 1000 bp merged peak intervals as binary training labels for its dense-epigenomic CNN) — https://github.com/bsml320/DeepFun
- **Basset** (Defines the 1000 bp CNN input window and the preprocessing pipeline (bedtools dependency) that DeepFun is built on) — https://github.com/davek44/Basset
- **bedtools** (Named dependency in the Basset README for BED interval data preprocessing (extension/sorting/merging of peak intervals)) — http://bedtools.readthedocs.io/en/latest/

## Examples

```
bedtools merge -d 200 -i sorted_extended_peaks.bed > merged_peaks.bed
```

## Evaluation signals

- Every output interval in merged_peaks.bed is exactly 1000 bp long
- No two intervals in merged_peaks.bed overlap by more than 200 bp
- Each merged peak center equals the weighted average of the midpoints of the constituent peaks in its cluster
- All rows are valid BED (at least chrom, start, end with 0 <= start < end and coordinates sorted by chromosome/start)
- Each input profile's redundant overlapping narrow-peak calls are collapsed within the same profile, so per-profile interval counts decrease after merging

## Limitations

- Training treats peaks as binary labels, ignoring the quantitative epigenomic signal intensity
- No normalization approach has yet been established to eliminate technical bias when integrating quantitative signals
- The interval handling is tied to the Basset framework (Torch7 and Python) input conventions; centers become weighted midpoints rather than the max-signal summit
- DeepFun's model does not consider long-range interactions such as 3D DNA contact, so interval context is local sequence only

## Evidence

- [readme] bedtools dependency: [evidence span withheld — anchor status: not_found]
- [discussion] binary peak labels during training: [evidence span withheld — anchor status: not_found]
- [abstract] ENCODE/Roadmap source profiles: "we collected chromatin profiles from ENCODE and Roadmap projects to construct the feature space"
- [methods] Basset preprocessing dependencies: "please follow Basset tutorials install dependencies packages at first."
- [readme] sequence-activity models the intervals train: [evidence span withheld — anchor status: external_doc]
