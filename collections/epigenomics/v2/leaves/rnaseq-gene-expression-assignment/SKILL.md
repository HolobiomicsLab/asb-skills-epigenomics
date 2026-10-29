---
name: rnaseq-gene-expression-assignment
description: Use when sgRNA or other genomic target sites need continuous epigenetic
  feature values (RNA expression from RNA-seq/GEO, accessibility from ATACdb, TSS
  distance) attached per locus before deep learning input.
when_to_use_negative:
- Input is already a per-sgRNA feature table with normalized epigenetic columns (only
  sequence encoding or model prediction remains)
- Only sequence-based prediction is required (EpiCas-DL offers seq-only models; position-based
  epigenetic assignment is unnecessary)
- The relevant epigenetic tracks are unavailable for the queried cell line (e.g.,
  CRISPRoff data limited to HEK293T and K562)
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: pybedtools
  role: Chromosome-position-based overlap to assign RNA expression, methylation, accessibility,
    and TSS-distance values to each sgRNA target site
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Host language for custom feature-assignment scripts
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: Downstream consumer of the min–max normalized epigenetic feature matrix (epi
    models)
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:3161d12a13715024f7a44206d36baa913cb25deff089a23570fb4108bf17dc38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python
  - Python 3.6
  - pybedtools
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# RNA-seq gene expression assignment

## Summary

Assigns gene expression (RNA-seq) values to genomic sgRNA target sites—alongside chromatin accessibility, methylation, and TSS-distance features—by chromosome-position overlap in pybedtools, then min–max normalizes all channels for model input.

## When to use

Use when sgRNA or other genomic target sites need continuous epigenetic feature values (RNA expression from RNA-seq/GEO, accessibility from ATACdb, TSS distance) attached per locus before deep learning input.

## When NOT to use

- Input is already a per-sgRNA feature table with normalized epigenetic columns (only sequence encoding or model prediction remains)
- Only sequence-based prediction is required (EpiCas-DL offers seq-only models; position-based epigenetic assignment is unnecessary)
- The relevant epigenetic tracks are unavailable for the queried cell line (e.g., CRISPRoff data limited to HEK293T and K562)

## Inputs

- sgRNA genomic coordinates (chrom, start, end, PAM position, transcript, gene) from CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1)
- FANTOM-derived TSS position table (primary and secondary CAGE TSS)
- Genome-wide ATAC-seq tracks from ATACdb (e.g., Sample_0101)
- WGBS DNA methylation data from GEO accessions (Table S2)
- RNA-seq gene expression data from GEO accessions (Table S2)

## Outputs

- Feature matrix with one row per sgRNA and seven min–max normalized epigenetic columns (4 nucleosome positioning, 1 accessibility, 1 methylation, 1 expression), all scaled to [0,1]

## How to apply

Load sgRNA genomic coordinates from the CRISPRoff/CRISPRi/CRISPRa screening datasets and a FANTOM-derived TSS position table, then compute distances to the primary and secondary CAGE TSS for each locus. Overlap each sgRNA chromosome position with genome-wide ATAC-seq tracks from ATACdb, and assign DNA methylation from WGBS data plus gene expression from RNA-seq at GEO accessions, performing all position-based feature assignment with pybedtools (v0.9.0) in Python 3.6. Min–max normalize all seven epigenetic channels (4 nucleosome-positioning distances, 1 accessibility, 1 methylation, 1 expression) so values fall in [0,1] before concatenating with one-hot encoded sgRNA sequences. The rationale is that EpiCas-DL showed methylation, accessibility, and expression level each stratify editing efficiency (e.g., highly expressed genes showed higher repression/activation efficiency).

## Related tools

- **pybedtools** (Chromosome-position-based overlap to assign RNA expression, methylation, accessibility, and TSS-distance values to each sgRNA target site) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Host language for custom feature-assignment scripts) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Downstream consumer of the min–max normalized epigenetic feature matrix (epi models)) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Output matrix contains exactly one row per sgRNA in the screening dataset
- All seven epigenetic feature columns are present and non-null per sgRNA
- After min–max normalization, every feature value lies within [0, 1]
- Nucleosome-positioning features equal the four primary/secondary CAGE TSS distance values per sgRNA
- Accessibility/methylation/expression values match the ATACdb track and GEO accession used for the given cell line (e.g., Hek293t)

## Limitations

- EpiCas-DL performance was limited by incomplete epigenetic feature information; only nucleosome positioning, accessibility, DNA methylation, and RNA expression are considered
- Epigenetic reference tracks are cell-line specific; the README distributes an epi_reference folder externally (Baidu link), so availability may constrain which cell lines can be annotated
- CRISPRoff screening data are mainly from HEK293T and K562, limiting generalization of the annotated features to other cell types

## Evidence

- [methods] custom scripts written in Python 3.6 with “pybedtools” (v0.9.0): "custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system"
- [methods] min–max normalization of epigenetic features: "All the epigenetic features were min–max normalized before model input"
- [methods] ATACdb as accessibility source: "genome-wide ATAC-seq data collected from the chromatin accessibility database ATACdb [24]"
- [readme] README: four epigenetic feature types: "combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
