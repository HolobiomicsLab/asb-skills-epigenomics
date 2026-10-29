---
name: atac-seq-tn5-offset-correction
description: Use when processing paired-end ATAC-seq alignments (e.g. GEO GSE66386)
  into fragment midpoint-count observables, where raw insert sizes must be shortened
  by 8 bp for the Tn5 9-bp duplication before classifying fragments.
when_to_use_negative:
- Input is MNase-seq data (e.g. GSE153609) — the −8 bp Tn5 offset is specific to ATAC-seq
  and must not be applied to MNase-seq fragment lengths
- Data are single-end ATAC-seq reads — fragment insert sizes cannot be computed from
  paired-end alignments as required
- The goal is chromatin occupancy profiling via another tool (e.g. NucleoATAC or MACS2+FIMO)
  whose preprocessing does not use RoboCOP's nucFrags/shortFrags midpoint tracks
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RoboCOP
  role: Analysis pipeline that loads the ATAC-seq BAM, applies the −8 bp Tn5 offset,
    classifies fragments, and builds the midpoint-count signals used as HMM observables
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:b71aa04a97a48e8a1ae9902c02a9b35f1b76703e6d6a476192ae2894174ca613
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ATAC-seq Tn5 offset correction

## Summary

Corrects paired-end ATAC-seq fragment lengths by subtracting 8 bp to compensate for the Tn5 transposase 9-bp insertion duplication before fragment-length classification and midpoint-count signal generation for RoboCOP.

## When to use

Use when processing paired-end ATAC-seq alignments (e.g. GEO GSE66386) into fragment midpoint-count observables, where raw insert sizes must be shortened by 8 bp for the Tn5 9-bp duplication before classifying fragments.

## When NOT to use

- Input is MNase-seq data (e.g. GSE153609) — the −8 bp Tn5 offset is specific to ATAC-seq and must not be applied to MNase-seq fragment lengths
- Data are single-end ATAC-seq reads — fragment insert sizes cannot be computed from paired-end alignments as required
- The goal is chromatin occupancy profiling via another tool (e.g. NucleoATAC or MACS2+FIMO) whose preprocessing does not use RoboCOP's nucFrags/shortFrags midpoint tracks

## Inputs

- Aligned paired-end ATAC-seq fragment alignments (BAM), e.g. GEO accession GSE66386
- RoboCOP configuration file (config.ini) with librobocop.so path set

## Outputs

- Tn5-offset-corrected fragment lengths used for nucFrags/shortFrags classification (127–187 bp and <80 bp thresholds)
- nucFrags and shortFrags midpoint-count tracks at single-nucleotide resolution, exported as bedGraph (fragment_midpoint_signals.bedgraph)

## How to apply

Load aligned paired-end fragment alignments (BAM) from the ATAC-seq dataset (GEO accession GSE66386) using the RoboCOP analysis pipeline. Before computing the fragment midpoint signal, subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp insertion duplication; this offset applies only to ATAC-seq, not to MNase-seq (e.g. GSE153609). After correction, partition fragments by insert size: fragments within 157±30 bp (127–187 bp) are labeled nucFrags and fragments shorter than 80 bp are labeled shortFrags. Compute each fragment's midpoint ((start+end)/2) and accumulate midpoint counts per genomic base at single-nucleotide resolution, producing one 1-D track per fragment class, and export as bedGraph files.

## Related tools

- **RoboCOP** (Analysis pipeline that loads the ATAC-seq BAM, applies the −8 bp Tn5 offset, classifies fragments, and builds the midpoint-count signals used as HMM observables) — https://github.com/HarteminkLab/RoboCOP

## Evaluation signals

- Verify fragment_midpoint_signals.bedgraph exists and contains both nucFrags and shortFrags tracks
- Confirm the −8 bp offset was applied before classification: corrected insert sizes (not raw) fall within the 157±30 bp nucFrags window and the <80 bp shortFrags cutoff
- Check midpoint tracks are 1-D count signals at single-nucleotide resolution, one per fragment class
- Confirm the offset was not applied to MNase-seq inputs (GSE153609), only to ATAC-seq (GSE66386)

## Limitations

- The −8 bp correction is specific to the Tn5 transposase 9-bp insertion duplication in ATAC-seq; applying it to MNase-seq would misclassify fragment lengths
- RoboCOP observes markedly better performance with MNase-seq than with ATAC-seq, due to genome-wide fragment distribution and MNase exonuclease activity reflecting DBF sizes, so ATAC-seq-derived signals are inherently less informative

## Evidence

- [methods] subtract 8 bp from each fragment length to compensate for the Tn5 transposase 9-bp insertion duplication: [evidence span withheld — anchor status: not_found]
- [supplementary] ATAC-seq data can be downloaded from GEO under accession number GSE66386: "ATAC-seq data can be downloaded from GEO under accession number GSE66386"
- [discussion] we observe markedly better performance when using MNase-seq than when using ATAC-seq: "we observe markedly better performance when using MNase-seq than when using ATAC-seq, for two primary reasons"
- [readme] chromatin accessibility data (currently uses only with MNase-seq or ATAC-seq): "chromatin accessibility data (currently uses only with MNase-seq or ATAC-seq) to compute genome-wide probabilistic occupancy landscape"
