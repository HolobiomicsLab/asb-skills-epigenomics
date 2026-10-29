---
name: mnase-seq-fragment-length-classification
description: Use when raw paired-end MNase-seq or ATAC-seq alignments (BAM) must be
  converted into fragment-length-classified midpoint count tracks (bedGraph) to feed
  a chromatin occupancy model such as RoboCOP.
when_to_use_negative:
- Input is single-end read data with no paired-end fragment insert sizes, since classification
  relies entirely on fragment length (start/end pairs)
- The goal is downstream occupancy inference itself (HMM decoding) rather than producing
  the observable signal tracks that precede it
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Analysis pipeline used to load alignments, classify fragments by insert size,
    compute midpoint count signals, and export bedGraph observables for its multivariate
    HMM
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:073b43f1aaa7b7127b3c0dfb2f564d0d8aa326da0bfd8edf4cbbf0cea5b94eec
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mnase-seq-fragment-length-classification

## Summary

Classifies paired-end chromatin accessibility fragments by insert size into nucleosome-sized (nucFrags) and short (shortFrags) classes and aggregates per-base midpoint counts into 1-D genomic signals used as HMM observables by RoboCOP.

## When to use

Use when raw paired-end MNase-seq or ATAC-seq alignments (BAM) must be converted into fragment-length-classified midpoint count tracks (bedGraph) to feed a chromatin occupancy model such as RoboCOP.

## When NOT to use

- Input is single-end read data with no paired-end fragment insert sizes, since classification relies entirely on fragment length (start/end pairs)
- The goal is downstream occupancy inference itself (HMM decoding) rather than producing the observable signal tracks that precede it

## Inputs

- Aligned paired-end fragment alignments (BAM) from MNase-seq (GEO GSE153609)
- Aligned paired-end fragment alignments (BAM) from ATAC-seq (GEO GSE66386)

## Outputs

- nucFrags midpoint-count bedGraph track (single-nucleotide resolution)
- shortFrags midpoint-count bedGraph track (single-nucleotide resolution)
- Combined fragment_midpoint_signals.bedgraph containing both tracks

## How to apply

Load aligned paired-end fragments from the MNase-seq dataset (GEO GSE153609) via the RoboCOP analysis pipeline and partition by insert size: fragments within 157±30 bp (127–187 bp) become nucFrags, fragments shorter than 80 bp become shortFrags. For each fragment compute the midpoint ((start+end)/2) and accumulate midpoint counts per genomic base into a single-nucleotide-resolution 1-D track, one track per fragment class. The rationale is that MNase cuts reflect the size of the DNA-binding factor, so nucleosome-sized and sub-nucleosomal fragment classes carry separable occupancy information that the multivariate HMM consumes as observables. For ATAC-seq data (GEO GSE66386), subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp duplication before computing the analogous midpoint signal. Export both tracks as bedGraph (e.g. fragment_midpoint_signals.bedgraph) and verify the classification thresholds and the −8 bp ATAC offset were applied.

## Related tools

- **RoboCOP** (Analysis pipeline used to load alignments, classify fragments by insert size, compute midpoint count signals, and export bedGraph observables for its multivariate HMM) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
git clone https://github.com/HarteminkLab/RoboCOP.git && cd RoboCOP && bash gccCompile
```

## Evaluation signals

- fragment_midpoint_signals.bedgraph exists and contains both nucFrags and shortFrags tracks
- Fragment-length thresholds used for classification match 157±30 bp (127–187 bp) for nucFrags and <80 bp for shortFrags
- ATAC-seq fragment lengths have the −8 bp Tn5 offset applied before midpoint computation (vs. MNase-seq with no offset)
- Midpoint counts are accumulated per genomic base at single-nucleotide resolution, one track per fragment class

## Limitations

- MNase-seq yields markedly better RoboCOP performance than ATAC-seq because genome-wide fragment distribution and MNase exonuclease activity reflect DBF sizes; ATAC-based signals are noisier and require the Tn5 offset correction
- Accuracy of the resulting occupancy profiles improves with deeper sequencing; shallow libraries degrade the signals

## Evidence

- [supplementary] fragments with length within 157±30 bp (127–187 bp) are labeled nucFrags, and fragments shorter than 80 bp are labeled shortFrags: [evidence span withheld — anchor status: not_found]
- [supplementary] The two-dimensional fragment representation is simplified into two one-dimensional genomic signals by separately aggregating the midpoint counts of nucFrags and shortFrags: [evidence span withheld — anchor status: not_found]
