---
name: epigenetic-feature-minmax-normalization
description: Use when building model inputs from sgRNA genomic coordinates and you
  need chromosome-position-based epigenetic annotations (ATACdb accessibility, FANTOM
  TSS distances, WGBS methylation, GEO RNA-seq expression) scaled to a common range.
when_to_use_negative:
- Input is already a normalized epigenetic feature matrix — re-annotating would duplicate
  feature assignment
- Task only requires sequence-based prediction; EpiCas-DL supports sequence-only mode
  (CRISPRoff_seq/CRISPRa_seq) without epigenetic features
- Cell-type-matched ATACdb/methylation/expression tracks do not exist for the target
  cell line, since the epigenetic features are cell-type dependent
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: pybedtools
  role: Bed-based chromosome-position overlap and feature assignment (accessibility,
    methylation, expression, TSS distances) to each sgRNA locus
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Host language for the custom annotation and min–max normalization scripts
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: Downstream deep learning model that consumes the min–max normalized epigenetic
    features together with one-hot encoded sgRNA sequences
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:9208604bfc8ccc57069642dffcd9b3fefb315072c25318fa6b007b1216bbeeaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python
  - Python 3.6
  - pybedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# epigenetic-feature-minmax-normalization

## Summary

Assigns seven continuous epigenetic features (4 nucleosome-positioning/TSS distances, 1 chromatin accessibility, 1 DNA methylation, 1 RNA expression) to each sgRNA target site and min–max normalizes them to [0,1] so they can be fed alongside one-hot encoded sgRNA sequences into EpiCas-DL.

## When to use

Use when building model inputs from sgRNA genomic coordinates and you need chromosome-position-based epigenetic annotations (ATACdb accessibility, FANTOM TSS distances, WGBS methylation, GEO RNA-seq expression) scaled to a common range.

## When NOT to use

- Input is already a normalized epigenetic feature matrix — re-annotating would duplicate feature assignment
- Task only requires sequence-based prediction; EpiCas-DL supports sequence-only mode (CRISPRoff_seq/CRISPRa_seq) without epigenetic features
- Cell-type-matched ATACdb/methylation/expression tracks do not exist for the target cell line, since the epigenetic features are cell-type dependent

## Inputs

- sgRNA genomic coordinates (chrom, start, end, PAM coordinate) from CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1)
- FANTOM-derived CAGE TSS position table (primary and secondary TSS)
- Genome-wide ATAC-seq tracks from ATACdb (e.g. Sample_0101)
- WGBS DNA methylation tracks and RNA-seq expression values from GEO accessions (Table S2)
- Cell-line specification (e.g. Hek293t, K562) for cell-type-matched tracks

## Outputs

- Per-sgRNA matrix row with 7 epigenetic feature columns (4 nucleosome positioning/TSS distances, 1 accessibility, 1 methylation, 1 expression)
- Min–max normalized feature matrix with all values scaled to [0, 1], ready for concatenation with one-hot encoded sgRNA sequences

## How to apply

Load sgRNA genomic coordinates from CRISPRoff/CRISPRi/CRISPRa screening datasets (e.g. CRISPRoff_tiling with 111,682 sgRNAs) and FANTOM-derived TSS positions, then use Python 3.6 with pybedtools (v0.9.0 per the paper) to assign features by chromosome position. For each sgRNA locus compute distance to the primary and secondary CAGE TSS (yielding four nucleosome-positioning values), overlap the locus with genome-wide ATAC-seq tracks from ATACdb (e.g. Sample_0101), and pull WGBS methylation and RNA-seq expression values from the GEO accessions listed in Table S2. Min–max normalize all seven epigenetic channels before model input, since EpiCas-DL consumes these scaled features together with one-hot encoded 40-nt sgRNA sequences; the README's example.py requires -chrom, -start, -end, -pamCoord, -transcript, -gene and -cell_line arguments to locate the epigenetic values. Decide the cell-line-specific tracks to use per input (e.g. Hek293t, K562) because accessibility/methylation/expression are cell-type dependent.

## Related tools

- **pybedtools** (Bed-based chromosome-position overlap and feature assignment (accessibility, methylation, expression, TSS distances) to each sgRNA locus) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Host language for the custom annotation and min–max normalization scripts) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Downstream deep learning model that consumes the min–max normalized epigenetic features together with one-hot encoded sgRNA sequences) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Output matrix contains exactly one row per sgRNA with the 7 epigenetic feature columns present
- After min–max normalization, every feature column has values within [0, 1]
- Each sgRNA has four nucleosome-positioning values (primary and secondary CAGE TSS distances)
- Assigned accessibility values match ATACdb tracks for the specified cell line (e.g. Sample_0101) at the sgRNA's chromosome position
- Predictions from the _epi model run without missing-feature errors when the -chrom/-start/-end/-pamCoord/-gene/-cell_line arguments are supplied

## Limitations

- Epigenetic feature information is incomplete (only nucleosome positioning, accessibility, methylation and expression considered), which limited EpiCas-DL performance
- Cell-type specificity: screening datasets are mainly HEK293T and K562, so generalization to other cell lines is limited
- Training data amount was relatively small for CRISPRoff, increasing overfitting risk

## Evidence

- [methods] pybedtools assigns chromosome-position features: "custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system"
- [methods] min-max normalization of epigenetic features: "All the epigenetic features were min–max normalized before model input"
- [methods] ATACdb as accessibility source: "genome-wide ATAC-seq data collected from the chromatin accessibility database ATACdb"
- [readme] README epigenetic invocation arguments: "python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087"
- [readme] four epigenetic feature types: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
