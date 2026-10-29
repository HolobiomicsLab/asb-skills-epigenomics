---
name: genomic-coverage-track-generation
description: Use when paired-end MNase-seq or ATAC-seq BAM alignments must be converted
  into fragment-length-classified, midpoint-aggregated bedGraph count tracks to serve
  as HMM observables for chromatin occupancy profiling.
when_to_use_negative:
- Input is already a normalized coverage or count track (e.g. an existing bedGraph/bigWig)
  rather than raw paired-end fragment alignments
- Data are single-end reads, since fragment midpoints require paired-end start and
  end coordinates
- The goal is peak calling or footprinting (e.g. MACS2 plus FIMO), not constructing
  midpoint-count observables for a multivariate HMM
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RoboCOP
  role: Analysis pipeline used to load paired-end fragment alignments, classify fragments
    by insert size (157±30 bp nucFrags; <80 bp shortFrags), compute midpoints, and
    export bedGraph count tracks consumed by its HMM
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:f325902e5ced3f7ea232fd386607607fa1b8fb965219474ef3244a7973c4995f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# genomic-coverage-track-generation

## Summary

Generates single-nucleotide-resolution 1-D genomic count tracks from paired-end chromatin accessibility fragment alignments by partitioning fragments into nucleosomal and subnucleosomal classes and aggregating their midpoint counts, producing the multivariate observables consumed by RoboCOP's HMM.

## When to use

Use when paired-end MNase-seq or ATAC-seq BAM alignments must be converted into fragment-length-classified, midpoint-aggregated bedGraph count tracks to serve as HMM observables for chromatin occupancy profiling.

## When NOT to use

- Input is already a normalized coverage or count track (e.g. an existing bedGraph/bigWig) rather than raw paired-end fragment alignments
- Data are single-end reads, since fragment midpoints require paired-end start and end coordinates
- The goal is peak calling or footprinting (e.g. MACS2 plus FIMO), not constructing midpoint-count observables for a multivariate HMM

## Inputs

- Aligned paired-end MNase-seq fragment alignments (BAM) from GSE153609
- Aligned paired-end ATAC-seq fragment alignments (BAM) from GSE66386

## Outputs

- bedGraph file (e.g. fragment_midpoint_signals.bedgraph) containing nucFrags and shortFrags midpoint-count tracks
- Single-nucleotide-resolution 1-D genomic count signal per fragment class

## How to apply

Load aligned paired-end fragments (BAM) from the MNase-seq dataset (GEO accession GSE153609) using the RoboCOP analysis pipeline. Partition fragments by insert size: fragments within 157±30 bp (127–187 bp) are labeled nucFrags, and fragments shorter than 80 bp are labeled shortFrags, reflecting nucleosome-sized versus subnucleosomal (DBF-protected) fragments. For each classified fragment compute the midpoint ((start+end)/2) and accumulate midpoint counts per genomic base into a 1-D signal track at single-nucleotide resolution, one track per fragment class. For ATAC-seq data (GEO accession GSE66386), subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp insertion duplication before computing midpoints. Export the nucFrags and shortFrags midpoint-count tracks as bedGraph files for downstream RoboCOP HMM inference.

## Related tools

- **RoboCOP** (Analysis pipeline used to load paired-end fragment alignments, classify fragments by insert size (157±30 bp nucFrags; <80 bp shortFrags), compute midpoints, and export bedGraph count tracks consumed by its HMM) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
grep -c nucFrags fragment_midpoint_signals.bedgraph && grep -c shortFrags fragment_midpoint_signals.bedgraph
```

## Evaluation signals

- The exported fragment_midpoint_signals.bedgraph exists and contains both nucFrags and shortFrags tracks
- Fragment-length classification thresholds match the specified values: nucFrags within 157±30 bp (127–187 bp) and shortFrags below 80 bp
- For ATAC-seq inputs, fragment lengths were offset by −8 bp (Tn5 9-bp insertion duplication compensation) before midpoint computation
- Midpoint counts are accumulated per genomic base at single-nucleotide resolution, with each track position equal to (start+end)/2 of classified fragments
- Resulting tracks serve as valid multivariate observables for the RoboCOP HMM without further re-binning

## Limitations

- The 8-bp Tn5 offset applies only to ATAC-seq; applying it to MNase-seq data would be incorrect
- RoboCOP's occupancy profile accuracy improves with sequencing depth; shallow MNase-seq or ATAC-seq libraries will yield noisier midpoint-count tracks
- No version numbers for tools (RoboCOP, COMPETE, NucleoATAC, MACS2, FIMO) are stated in the paper, so reproducibility of exact track generation depends on repository state at the time of use

## Evidence

- [methods] fragments with length within 157±30 bp (127–187 bp) are labeled nucFrags, and fragments shorter than 80 bp are labeled shortFrags: [evidence span withheld — anchor status: not_found]
- [methods] For each classified fragment, compute the midpoint ((start+end)/2) and accumulate midpoint counts per genomic base into a 1-D signal track: [evidence span withheld — anchor status: not_found]
- [methods] For ATAC-seq alignments (GEO accession GSE66386), subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp insertion duplication: [evidence span withheld — anchor status: not_found]
- [methods] Export the nucFrags and shortFrags midpoint-count tracks as bedGraph files: [evidence span withheld — anchor status: not_found]
