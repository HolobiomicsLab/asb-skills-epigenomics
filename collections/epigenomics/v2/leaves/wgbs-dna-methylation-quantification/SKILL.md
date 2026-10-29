---
name: wgbs-dna-methylation-quantification
description: Use when each sgRNA locus needs a methylation value from WGBS/GEO tracks
  assigned by genomic coordinate overlap before min-max normalization and deep-learning
  model input.
when_to_use_negative:
- Input is already a feature table with methylation values assigned per sgRNA (no
  coordinate overlap needed)
- Prediction is sequence-only (EpiCas-DL seq models omit epigenetic features entirely)
- Cell type of the sgRNA screen has no matching WGBS track in the reference, since
  methylation is cell-type specific
edam_operation: http://edamontology.org/operation_3206
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: pybedtools
  role: Chromosome-position-based assignment of WGBS methylation values to each sgRNA
    locus
  repo: https://github.com/yangqianq/EpiCas-DL
- name: Python 3.6
  role: Custom scripts wrapping pybedtools for feature assignment and min-max normalization
  repo: https://github.com/yangqianq/EpiCas-DL
- name: EpiCas-DL
  role: Downstream deep-learning model consuming the methylation channel together
    with six other epigenetic features and one-hot encoded sgRNA sequences
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:6f784cf9c51204774d3930e3b3e0f1b4956a43de1d1ae582a567fa821c1ccbbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python
  - Python 3.6
  - pybedtools
  edam_operation: http://edamontology.org/operation_3206
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# wgbs-dna-methylation-quantification

## Summary

Assigns WGBS-derived DNA methylation values (GEO accessions listed in Table S2) to sgRNA target sites by chromosome-position overlap using pybedtools, as one of seven epigenetic feature channels for EpiCas-DL sgRNA activity prediction.

## When to use

Use when each sgRNA locus needs a methylation value from WGBS/GEO tracks assigned by genomic coordinate overlap before min-max normalization and deep-learning model input.

## When NOT to use

- Input is already a feature table with methylation values assigned per sgRNA (no coordinate overlap needed)
- Prediction is sequence-only (EpiCas-DL seq models omit epigenetic features entirely)
- Cell type of the sgRNA screen has no matching WGBS track in the reference, since methylation is cell-type specific

## Inputs

- sgRNA genomic coordinates (chromosome, start, end) from CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1)
- WGBS DNA methylation genome tracks from GEO accessions listed in Table S2
- Cell line / gene / transcript annotation per sgRNA (e.g. chr12:100592057-100592096, gene ACTR6, cell line Hek293t)

## Outputs

- Per-sgRNA DNA methylation value column within the 7-channel epigenetic feature matrix
- Min-max normalized methylation values scaled to [0, 1] for model input

## How to apply

Load sgRNA genomic coordinates from the CRISPRoff/CRISPRi/CRISPRa screening datasets (Table S1) as BED-like intervals, and the WGBS methylation tracks from the GEO accessions in Table S2. Use pybedtools (v0.9.0) in Python 3.6 to overlap each sgRNA chromosome position with the WGBS track and assign the methylation value at that locus. Combine methylation with the other six channels (4 nucleosome-positioning distances from FANTOM CAGE TSSs, 1 ATACdb accessibility, 1 RNA-seq expression), then min-max normalize all seven epigenetic features before feeding them, together with one-hot encoded sgRNA sequences, into EpiCas-DL. The rationale is that endogenous DNA methylation improves silencing: target sites with DNA methylation showed higher gene repression efficiency in the CRISPRoff datasets.

## Related tools

- **pybedtools** (Chromosome-position-based assignment of WGBS methylation values to each sgRNA locus) — https://github.com/yangqianq/EpiCas-DL
- **Python 3.6** (Custom scripts wrapping pybedtools for feature assignment and min-max normalization) — https://github.com/yangqianq/EpiCas-DL
- **EpiCas-DL** (Downstream deep-learning model consuming the methylation channel together with six other epigenetic features and one-hot encoded sgRNA sequences) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- Output matrix contains one row per sgRNA with the methylation column present among the 7 epigenetic feature columns
- All methylation values are min-max normalized to [0, 1] before model input
- Each sgRNA's methylation value matches the WGBS track value at its exact chromosome position when spot-checked in a genome browser
- Target sites flagged as methylated show higher predicted/observed repression efficiency in CRISPRoff datasets, reproducing the paper's Fig. S2a trend
- pybedtools version pinned (v0.9.0 in the article; v0.8.2 in the README requirements) so overlap results are reproducible

## Limitations

- Methylation tracks are cell-type specific; only HEK293T and K562 CRISPRoff screens are well covered, limiting generalization
- Incomplete epigenetic feature information (including methylation coverage gaps) limited EpiCas-DL model performance
- WGBS reference tracks must be downloaded separately (the README's 'epi_reference' folder is hosted on an external pan.baidu.com link, not in the repo)

## Evidence

- [methods] Assign DNA methylation values from WGBS data and gene expression values from RNA-seq data at the GEO accessions listed in Table S2, using pybedtools (v0.9.0) in Python: [evidence span withheld — anchor status: not_found]
- [results] sites with DNA methylation showed higher gene repression efficiency in the two CRISPRoff datasets (Fig. S2a), suggesting that endogenous DNA methylation could improve the silencing effects: "sites with DNA methylation showed higher gene repression efficiency in the two CRISPRoff datasets (Fig. S2a), suggesting that endogenous DNA methylation could improve the silencing effects"
- [readme] The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "The tool combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
