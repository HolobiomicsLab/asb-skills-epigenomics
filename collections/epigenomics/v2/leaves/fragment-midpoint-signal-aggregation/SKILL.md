---
name: fragment-midpoint-signal-aggregation
description: Use when paired-end MNase-seq or ATAC-seq fragment alignments (BAM) must
  be classified by insert size and converted into single-nucleotide-resolution midpoint
  count tracks (bedGraph) to feed RoboCOP's HMM.
when_to_use_negative:
- Input is single-end sequencing data without paired-end fragment start/end coordinates,
  since midpoints cannot be computed
- The chromatin accessibility data are already reduced to occupancy or peak calls
  rather than raw paired-end fragments
- The downstream model does not consume per-base midpoint count observables (e.g.
  peak-calling workflows such as MACS2 followed by FIMO labeling)
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0092
tools:
- name: RoboCOP
  role: Analysis pipeline within which fragments are classified (nucFrags 127–187
    bp, shortFrags <80 bp), midpoints accumulated, and midpoint-count bedGraph tracks
    produced as HMM observables
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:f63aab1b94ad2fea284b587e07e59ea7b07a7880b3bd61b6cb924bb94e7acf30
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fragment-midpoint-signal-aggregation

## Summary

Reduces paired-end chromatin accessibility fragments (MNase-seq or ATAC-seq) into two 1-D genomic count signals by classifying fragments by insert size (nucFrags vs. shortFrags) and aggregating per-base fragment midpoint counts, which serve as the multivariate observables consumed by RoboCOP's HMM.

## When to use

Use when paired-end MNase-seq or ATAC-seq fragment alignments (BAM) must be classified by insert size and converted into single-nucleotide-resolution midpoint count tracks (bedGraph) to feed RoboCOP's HMM.

## When NOT to use

- Input is single-end sequencing data without paired-end fragment start/end coordinates, since midpoints cannot be computed
- The chromatin accessibility data are already reduced to occupancy or peak calls rather than raw paired-end fragments
- The downstream model does not consume per-base midpoint count observables (e.g. peak-calling workflows such as MACS2 followed by FIMO labeling)

## Inputs

- Aligned paired-end fragment alignments (BAM) from MNase-seq (e.g. GEO GSE153609)
- Aligned paired-end fragment alignments (BAM) from ATAC-seq (e.g. GEO GSE66386)
- Fragment insert-size classification thresholds (157±30 bp for nucFrags; <80 bp for shortFrags; −8 bp offset for ATAC-seq)

## Outputs

- nucFrags per-base midpoint-count signal track (bedGraph)
- shortFrags per-base midpoint-count signal track (bedGraph)
- fragment_midpoint_signals.bedgraph containing both nucFrags and shortFrags tracks

## How to apply

Load aligned paired-end fragment alignments (BAM) — e.g. MNase-seq from GEO accession GSE153609 or ATAC-seq from GSE66386 — using the RoboCOP analysis pipeline. Partition fragments by insert size: fragments within 157±30 bp (127–187 bp) are labeled nucFrags, and fragments shorter than 80 bp are labeled shortFrags, reflecting nucleosome-sized versus DBF-sized protection. For each classified fragment compute the midpoint ((start+end)/2) and accumulate midpoint counts per genomic base into a 1-D signal track at single-nucleotide resolution, one track per fragment class. For ATAC-seq data, first subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp insertion duplication before computing the analogous midpoint signal. Export the nucFrags and shortFrags midpoint-count tracks as bedGraph files for downstream HMM inference.

## Related tools

- **RoboCOP** (Analysis pipeline within which fragments are classified (nucFrags 127–187 bp, shortFrags <80 bp), midpoints accumulated, and midpoint-count bedGraph tracks produced as HMM observables) — https://github.com/HarteminkLab/RoboCOP

## Evaluation signals

- fragment_midpoint_signals.bedgraph exists and contains both nucFrags and shortFrags tracks
- Fragment-length thresholds used for classification match 157±30 bp (127–187 bp) for nucFrags and <80 bp for shortFrags
- The −8 bp ATAC-seq Tn5 offset is applied before computing midpoints for ATAC-seq alignments
- Tracks are aggregated at single-nucleotide resolution with counts accumulated at each fragment midpoint ((start+end)/2)
- Tracks export cleanly as bedGraph files consumable as multivariate observables by RoboCOP's HMM

## Limitations

- The thresholds and fragment classes are specific to yeast (S. cerevisiae) MNase-seq/ATAC-seq assays as used in the RoboCOP study
- RoboCOP currently uses only MNase-seq or ATAC-seq accessibility data, and performance is markedly better with MNase-seq due to genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes
- Accuracy of downstream occupancy profiles improves with deeper sequencing; sequencing depth requirements are not quantified

## Evidence
