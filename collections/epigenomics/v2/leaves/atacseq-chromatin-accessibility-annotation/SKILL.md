---
name: atacseq-chromatin-accessibility-annotation
description: Use when sgRNA genomic coordinates must be assigned chromatin accessibility
  values from genome-wide ATAC-seq tracks (ATACdb) as epigenetic features for sgRNA
  activity prediction.
when_to_use_negative:
- Prediction is sequence-only (e.g., the CRISPRoff_seq model path) and no genomic
  coordinates or epigenetic features are available
- The target cell line has no matching ATACdb ATAC-seq sample, so accessibility cannot
  be validly assigned to sgRNA loci
- Input is already a normalized epigenetic feature matrix; in that case proceed directly
  to min-max normalization check and model input
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: pybedtools
  role: Performs chromosome-position-based overlap of sgRNA coordinates with ATACdb
    ATAC-seq tracks to assign accessibility values
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Hosts the custom pybedtools annotation scripts for feature assignment
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: 'Downstream consumer: the accessibility channel is min-max normalized and
    fed with one-hot encoded sgRNA sequences into the model'
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:1158b0f8bf3a9ffa28ad66c372d03da0690fbf8b9a916e66cc5a1cfb5a63084f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python
  - Python 3.6
  - pybedtools
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# atacseq-chromatin-accessibility-annotation

## Summary

Annotates sgRNA target sites with chromatin accessibility from ATACdb (e.g., Sample_0101) using pybedtools (v0.9.0) chromosome-position overlap, as one of seven epigenetic feature channels feeding EpiCas-DL.

## When to use

Use when sgRNA genomic coordinates must be assigned chromatin accessibility values from genome-wide ATAC-seq tracks (ATACdb) as epigenetic features for sgRNA activity prediction.

## When NOT to use

- Prediction is sequence-only (e.g., the CRISPRoff_seq model path) and no genomic coordinates or epigenetic features are available
- The target cell line has no matching ATACdb ATAC-seq sample, so accessibility cannot be validly assigned to sgRNA loci
- Input is already a normalized epigenetic feature matrix; in that case proceed directly to min-max normalization check and model input

## Inputs

- sgRNA genomic coordinate table (chromosome, start, end) from CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1)
- genome-wide ATAC-seq chromatin accessibility tracks from ATACdb (Sample_0101), cell-line matched

## Outputs

- Per-sgRNA chromatin accessibility value as one column of the 7-channel epigenetic feature matrix
- Min-max normalized epigenetic feature matrix (values in [0, 1]) with one row per sgRNA

## How to apply

Load sgRNA genomic coordinates (chromosome, start, end) from the CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1), then overlap each sgRNA locus with genome-wide ATAC-seq tracks from ATACdb Sample_0101 using pybedtools (v0.9.0) custom scripts in Python 3.6. Assign the overlapping accessibility value to each sgRNA as one feature channel alongside nucleosome positioning (4 CAGE-TSS distances from FANTOM), DNA methylation (WGBS), and RNA expression (RNA-seq from GEO accessions in Table S2). Min-max normalize all seven epigenetic feature channels to [0, 1] before concatenating with one-hot encoded sgRNA sequences as EpiCas-DL model input. The rationale is that target sites with high chromatin accessibility showed higher editing efficiency in all examined datasets, so accessibility improves sgRNA efficacy prediction beyond sequence-only models.

## Related tools

- **pybedtools** (Performs chromosome-position-based overlap of sgRNA coordinates with ATACdb ATAC-seq tracks to assign accessibility values) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Hosts the custom pybedtools annotation scripts for feature assignment) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Downstream consumer: the accessibility channel is min-max normalized and fed with one-hot encoded sgRNA sequences into the model) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Output matrix contains exactly one row per sgRNA with all 7 epigenetic feature columns present
- All epigenetic feature values fall within [0, 1] after min-max normalization
- Each sgRNA's accessibility value derives from a valid overlap with ATACdb Sample_0101 tracks on the matching chromosome
- Accessibility channel separates expected biology: high-accessibility target sites should correspond to higher editing efficiency (Fig. S2c trend)
- Feature assignment is reproducible with pybedtools v0.9.0 scripts on the documented CentOS environment

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are considered; incomplete epigenetic feature information limited EpiCas-DL model performance
- Screening datasets are mainly in HEK293T and K562 cell lines, so ATACdb accessibility from other cell types may increase overfitting risk and limit generalization
- pybedtools version differs between paper (v0.9.0) and repository README (0.8.2); verify environment consistency

## Evidence

- [methods] Assign chromatin accessibility to each sgRNA by overlapping its chromosome position with genome-wide ATAC-seq tracks from ATACdb Sample_0101.: [evidence span withheld — anchor status: not_found]
- [methods] custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system: "custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system"
- [results] the target sites with high chromatin accessibility also revealed higher editing efficiency than those with low accessibility in all the examined datasets: "the target sites with high chromatin accessibility also revealed higher editing efficiency than those with low accessibility in all the examined datasets (Fig. S2c)"
- [readme] EpiCas-DL is a deeplearning based prediction tool ... combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
