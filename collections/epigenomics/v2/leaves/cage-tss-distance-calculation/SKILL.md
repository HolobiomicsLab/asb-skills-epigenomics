---
name: cage-tss-distance-calculation
description: Use when you have sgRNA genomic coordinates (e.g. from CRISPRoff/CRISPRi/CRISPRa
  screening tables) and FANTOM CAGE TSS positions, and need TSS-proximity/nucleosome-positioning
  features per sgRNA for a model or filter.
when_to_use_negative:
- You already have a feature matrix containing TSS-distance or nucleosome-positioning
  columns and only need downstream modeling or normalization.
- Your input lacks genomic coordinates for sgRNAs (sequence-only input); EpiCas-DL
  supports sequence-only prediction via CRISPRoff_seq/CRISPRa_seq models.
- Cell type or gene is unknown, so primary/secondary CAGE TSS cannot be correctly
  matched to the target locus.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: pybedtools
  role: Python wrapper over BEDTools used for all chromosome-position-based feature
    assignment, including computing sgRNA-to-CAGE-TSS distances
- name: EpiCas-DL
  role: Deep learning framework that consumes the CAGE-TSS distance features (example.py
    accepts -chrom, -start, -end, -pamCoord, -transcript, -gene, -cell_line)
  repo: https://github.com/yangqianq/EpiCas-DL
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:e407764f1a04c8199171593c0d6af470a2c7ae8917533578b1090f10251e1ea6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python
  - pybedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# cage-tss-distance-calculation

## Summary

Assigns each sgRNA target site its distance to FANTOM-derived primary and secondary CAGE transcription start sites (nucleosome positioning features) using pybedtools, as done in EpiCas-DL to enrich sgRNA activity prediction beyond sequence-only models.

## When to use

Use when you have sgRNA genomic coordinates (e.g. from CRISPRoff/CRISPRi/CRISPRa screening tables) and FANTOM CAGE TSS positions, and need TSS-proximity/nucleosome-positioning features per sgRNA for a model or filter.

## When NOT to use

- You already have a feature matrix containing TSS-distance or nucleosome-positioning columns and only need downstream modeling or normalization.
- Your input lacks genomic coordinates for sgRNAs (sequence-only input); EpiCas-DL supports sequence-only prediction via CRISPRoff_seq/CRISPRa_seq models.
- Cell type or gene is unknown, so primary/secondary CAGE TSS cannot be correctly matched to the target locus.

## Inputs

- sgRNA genomic coordinates (chrom, start, end, PAM position, strand) from CRISPRoff/CRISPRi/CRISPRa screening datasets (e.g. Table S1)
- FANTOM-derived TSS position table with primary and secondary CAGE TSS per gene
- gene/transcript identifiers and cell line context per sgRNA (e.g. ACTR6, transcript P1P2, Hek293t)

## Outputs

- Four nucleosome-positioning distance values per sgRNA (distance to primary and secondary CAGE TSS)
- One row per sgRNA in the epigenetic feature matrix (7 columns total when combined with accessibility, methylation and expression), values min–max scaled to [0, 1]

## How to apply

Load sgRNA genomic coordinates from the CRISPRoff/CRISPRi/CRISPRa screening datasets (e.g. CRISPRoff_tiling: 111,682 sgRNAs within ±2.5 kb of 520 genes' TSSs) together with the FANTOM-derived TSS position table. For each sgRNA locus, compute the distance to the primary CAGE TSS and the distance to the secondary CAGE TSS, yielding four nucleosome-positioning values per sgRNA (up/downstream of each TSS class). Perform all chromosome-position-based feature assignment with pybedtools (v0.9.0 in the paper's Python 3.6 scripts), and record the sgRNA's gene, transcript (e.g. P1P2), strand and cell line so TSS distances are matched to the correct annotation. Pass the resulting distance features through min–max normalization before model input, as EpiCas-DL does for all epigenetic channels.

## Related tools

- **pybedtools** (Python wrapper over BEDTools used for all chromosome-position-based feature assignment, including computing sgRNA-to-CAGE-TSS distances)
- **EpiCas-DL** (Deep learning framework that consumes the CAGE-TSS distance features (example.py accepts -chrom, -start, -end, -pamCoord, -transcript, -gene, -cell_line)) — https://github.com/yangqianq/EpiCas-DL

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- The output matrix contains one row per sgRNA with the 7 epigenetic feature columns present (4 nucleosome-positioning distances plus accessibility, methylation, expression).
- All feature values are scaled to [0, 1] after min–max normalization.
- Exactly four nucleosome-positioning values are produced per sgRNA, corresponding to primary and secondary CAGE TSS distances.
- Computed distances match recomputable coordinate arithmetic for spot-checked sgRNA loci (e.g. chr12:100592057-100592096, ACTR6).
- Feature assignment steps reproduce when re-run with pybedtools on the same sgRNA BED and FANTOM TSS inputs.

## Limitations

- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are considered; incomplete epigenetic feature information limited EpiCas-DL performance.
- CRISPRoff screening data were mainly from HEK293T and K562 cell lines, which may increase overfitting risk and limit generalization of the trained models.
- Differences between CRISPRoff and CRISPRi systems might affect prediction performance in testing datasets.

## Evidence

- [methods] For each sgRNA locus, compute the distance to the primary CAGE TSS and the distance to the secondary CAGE TSS, yielding four nucleosome-positioning values per sgRNA.: [evidence span withheld — anchor status: not_found]
- [methods] custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system: "custom scripts written in Python 3.6 with “pybedtools” (v0.9.0) on the CentOS system"
- [methods] All the epigenetic features were min–max normalized before model input: "All the epigenetic features were min–max normalized before model input"
- [readme] combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site: "combined four types of epigenetic features, including methylation, gene expression, chromatin accessibility and the distance between target site and transcription start site"
