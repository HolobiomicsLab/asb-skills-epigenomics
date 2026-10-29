---
name: sgrna-genomic-interval-assignment
description: Use when sgRNA loci from CRISPRoff/CRISPRi/CRISPRa screens must be assigned
  cell-type epigenetic values (accessibility, TSS distance, methylation, expression)
  by genomic interval overlap before model training or prediction.
when_to_use_negative:
- The input is already a normalized sgRNA-by-feature matrix ready for EpiCas-DL —
  re-running interval assignment would be redundant.
- 'Only sequence-based prediction is needed: EpiCas-DL''s seq-only models (CRISPRoff_seq/CRISPRa_seq)
  require just the one-hot encoded sgRNA sequence, no epigenetic annotation.'
- Cell-type-specific ATACdb/GEO tracks for the sgRNA's cell line are unavailable —
  assigned epigenetic values would be unreliable for that context.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: pybedtools
  role: Performs all chromosome-position-based feature assignment by overlapping sgRNA
    intervals with ATACdb accessibility, methylation, and expression tracks
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Hosts the custom feature-assignment scripts wrapping pybedtools on the CentOS
    system
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: Consumes the min-max normalized 7-channel epigenetic feature matrix together
    with one-hot encoded sgRNA sequences for activity prediction
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:fec73eb8d94aec44a09c4c9b507fe59cf643d20f843820e916c2bd70a26a2aaa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python
  - Python 3.6
  - pybedtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sgrna-genomic-interval-assignment

## Summary

Annotates each sgRNA target site with seven continuous epigenetic feature channels (4 nucleosome-positioning distances to primary/secondary CAGE TSS, ATACdb chromatin accessibility, WGBS DNA methylation, RNA-seq expression) using pybedtools-based chromosome-position overlap, followed by min-max normalization to [0, 1] for EpiCas-DL input.

## When to use

Use when sgRNA loci from CRISPRoff/CRISPRi/CRISPRa screens must be assigned cell-type epigenetic values (accessibility, TSS distance, methylation, expression) by genomic interval overlap before model training or prediction.

## When NOT to use

- The input is already a normalized sgRNA-by-feature matrix ready for EpiCas-DL — re-running interval assignment would be redundant.
- Only sequence-based prediction is needed: EpiCas-DL's seq-only models (CRISPRoff_seq/CRISPRa_seq) require just the one-hot encoded sgRNA sequence, no epigenetic annotation.
- Cell-type-specific ATACdb/GEO tracks for the sgRNA's cell line are unavailable — assigned epigenetic values would be unreliable for that context.

## Inputs

- sgRNA genomic coordinates (chromosome, start, end, PAM coordinate) from CRISPRoff_tiling, CRISPRoff_genome, CRISPRi_*, and hCRISPRa-v2 screening datasets (Table S1)
- FANTOM-derived TSS position table (primary and secondary CAGE TSS per gene)
- Genome-wide ATAC-seq tracks from ATACdb Sample_0101
- WGBS DNA methylation data (GEO accessions in Table S2)
- RNA-seq gene expression data (GEO accessions in Table S2)

## Outputs

- Feature matrix with one row per sgRNA and seven epigenetic feature columns (4 nucleosome-positioning distances, 1 accessibility, 1 methylation, 1 expression), all min-max scaled to [0, 1]

## How to apply

Load sgRNA genomic coordinates from the CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1) and the FANTOM-derived TSS position table. For each sgRNA locus, compute the distance to the primary CAGE TSS and to the secondary CAGE TSS, yielding four nucleosome-positioning values per sgRNA. Assign chromatin accessibility by overlapping each sgRNA's chromosome position with genome-wide ATAC-seq tracks from ATACdb (Sample_0101), then assign DNA methylation from WGBS and gene expression from RNA-seq at the GEO accessions in Table S2, performing all chromosome-position-based feature assignment with pybedtools (v0.9.0) in Python. Finally min-max normalize all seven epigenetic feature channels before feeding them, alongside one-hot encoded sgRNA sequences, into EpiCas-DL.

## Related tools

- **pybedtools** (Performs all chromosome-position-based feature assignment by overlapping sgRNA intervals with ATACdb accessibility, methylation, and expression tracks) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Hosts the custom feature-assignment scripts wrapping pybedtools on the CentOS system) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Consumes the min-max normalized 7-channel epigenetic feature matrix together with one-hot encoded sgRNA sequences for activity prediction) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- The output matrix contains exactly one row per sgRNA in the input screening dataset.
- All seven epigenetic feature columns (4 nucleosome positioning, 1 accessibility, 1 methylation, 1 expression) are present per row.
- Every feature value falls within [0, 1] after min-max normalization.
- TSS-distance columns yield four nucleosome-positioning values per sgRNA (primary and secondary CAGE TSS).
- Accessibility/methylation/expression values change consistently when the sgRNA is re-annotated against a different cell line's tracks (e.g., Hek293t vs K562).

## Limitations

- EpiCas-DL's performance was limited by incomplete epigenetic feature information; only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are considered.
- Available CRISPRoff screening datasets are relatively small and mainly from HEK293T and K562 cell lines, so annotations for other cell types risk overfitting and limited generalization.
- The epigenetic reference tracks are distributed via an external link (Baidu pan 'epi_reference' folder), not bundled in the repository.

## Evidence

- [methods] custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system: "custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system"
- [methods] genome-wide ATAC-seq data collected from the chromatin accessibility database ATACdb: "genome-wide ATAC-seq data collected from the chromatin accessibility database ATACdb [24]"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [readme] combined four types of epigenetic features: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
- [methods] One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme: "One-hot encoding strategy was used to formulate the sgRNA sequence into an image-like coding scheme"
