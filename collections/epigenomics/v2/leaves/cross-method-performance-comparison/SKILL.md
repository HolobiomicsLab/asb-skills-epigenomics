---
name: cross-method-performance-comparison
description: Use when a Source Data supplementary table must be re-analyzed to verify
  reported average accuracies across competing celltyping methods, e.g. the 7-task
  FACS-sorted PBMC targets in Fig 3a.
when_to_use_negative:
- The analysis goal is running celltyping on new scATAC-seq data rather than verifying
  or comparing reported benchmark accuracies — use Cellcano directly (pip install
  Cellcano) instead
- The target dataset is not one of the FACS-sorted gold-standard human PBMC tasks
  in Fig 3a (e.g. non-gold-standard targets or mouse brain tasks) — task subset selection
  differs and the reported 0.852/0.837/0.782 values will not apply
- Only one method's output is available and no cross-method comparison from the Source
  Data is possible
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Cellcano
  role: Benchmark-winning supervised scATAC-seq celltyping method whose two-round
    learning (MLP then self-Knowledge Distiller on low-entropy anchors) yields the
    highest average accuracy (0.852) being verified
  repo: https://marvinquiet.github.io/Cellcano/
- name: scJoint
  role: Compared label-transfer method (scRNA-seq reference to scATAC-seq target)
    that ranks second in the Fig 3a benchmark with mean accuracy 0.837
- name: ACTINN
  role: Compared supervised celltyping method ranking third in the Fig 3a benchmark
    with mean accuracy 0.782
- name: EpiAnno
  role: Recently published supervised scATAC-seq celltyping method included among
    the compared baselines in the benchmark
  repo: https://github.com/xy-chen16/EpiAnno
- name: pandas
  role: Used to load the Fig 3a sheet of the Source Data file, group accuracy rows
    by method, and compute the seven-task means
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:965d8669fb51a0e27354598af8d5d43a47180cd3dd1d4a8dedfae4e900aa5dea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ACTINN
  - Cellcano
  - EpiAnno
  - pandas
  - scJoint
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-method-performance-comparison

## Summary

Reproduce the supervised scATAC-seq celltyping benchmark from Fig. 3a of the Cellcano paper by recomputing per-method average prediction accuracy from the article's Source Data file and verifying it against reported values (Cellcano 0.852, scJoint 0.837, ACTINN 0.782 on seven FACS-sorted gold-standard human PBMC tasks).

## When to use

Use when a Source Data supplementary table must be re-analyzed to verify reported average accuracies across competing celltyping methods, e.g. the 7-task FACS-sorted PBMC targets in Fig 3a.

## When NOT to use

- The analysis goal is running celltyping on new scATAC-seq data rather than verifying or comparing reported benchmark accuracies — use Cellcano directly (pip install Cellcano) instead
- The target dataset is not one of the FACS-sorted gold-standard human PBMC tasks in Fig 3a (e.g. non-gold-standard targets or mouse brain tasks) — task subset selection differs and the reported 0.852/0.837/0.782 values will not apply
- Only one method's output is available and no cross-method comparison from the Source Data is possible

## Inputs

- Source Data supplementary Excel file from Nature Communications 10.1038/s41467-023-37439-3, specifically the sheet for Fig 3a
- Per-task prediction accuracy values for the seven FACS-sorted gold-standard human PBMC target celltyping tasks
- Reported benchmark values for verification (Cellcano 0.852, scJoint 0.837, ACTINN 0.782)

## Outputs

- fig3a_accuracy_summary.csv containing method-level average accuracies across the seven gold-standard tasks
- Deviation flags for any method whose recomputed mean differs from the reported value by more than 0.005

## How to apply

Download the Source Data supplementary file for Nature Communications article 10.1038/s41467-023-37439-3 and locate the sheet corresponding to Fig 3a. Load that sheet with pandas, extracting the per-task prediction accuracy values for the seven FACS-sorted gold-standard human PBMC target tasks for each method (Cellcano, scJoint, ACTINN, plus other compared methods such as Seurat, Signac, and EpiAnno). Group by method and compute the mean accuracy across the seven tasks; Cellcano's two-round supervised algorithm (MLP first round, self-Knowledge Distiller on low-entropy anchor cells second round) is expected to lead. Flag any method whose recomputed mean deviates from the reported value by more than 0.005 tolerance. Write the summary table of the three method-level average accuracies to fig3a_accuracy_summary.csv.

## Related tools

- **Cellcano** (Benchmark-winning supervised scATAC-seq celltyping method whose two-round learning (MLP then self-Knowledge Distiller on low-entropy anchors) yields the highest average accuracy (0.852) being verified) — https://marvinquiet.github.io/Cellcano/
- **scJoint** (Compared label-transfer method (scRNA-seq reference to scATAC-seq target) that ranks second in the Fig 3a benchmark with mean accuracy 0.837)
- **ACTINN** (Compared supervised celltyping method ranking third in the Fig 3a benchmark with mean accuracy 0.782)
- **EpiAnno** (Recently published supervised scATAC-seq celltyping method included among the compared baselines in the benchmark) — https://github.com/xy-chen16/EpiAnno
- **pandas** (Used to load the Fig 3a sheet of the Source Data file, group accuracy rows by method, and compute the seven-task means)

## Examples

```
import pandas as pd; df = pd.read_excel('SourceData_10.1038_s41467-023-37439-3.xlsx', sheet_name='Fig3a'); print(df.groupby('method')['accuracy'].mean())
```

## Evaluation signals

- Recomputed mean accuracy for Cellcano across the seven FACS-sorted gold-standard PBMC tasks equals 0.852 within the 0.005 tolerance
- Recomputed means for scJoint and ACTINN equal 0.837 and 0.782 respectively within 0.005
- Exactly seven tasks per method are included in the groupby aggregation, matching the FACS-sorted gold-standard target subset
- fig3a_accuracy_summary.csv is produced and preserves the paper's ranking: Cellcano > scJoint > ACTINN
- No method's mean deviates from its reported value by more than 0.005 (no flags raised)

## Limitations

- The benchmark covers 50 celltyping tasks from two systems (human PBMCs and mouse brain); the 0.852/0.837/0.782 averages apply only to the seven FACS-sorted gold-standard target tasks
- No specific versions were reported for the compared tools (Seurat, Signac, scJoint), so exact version-level reproduction is not possible
- Cellcano's compared scope is supervised celltyping with scATAC-seq reference; cross-modality prediction using scRNA-seq reference and multimodal reference data are not implemented
- Compared integration methods (Seurat, Signac, scJoint) jointly operate on reference and target datasets, so their runtimes and computational costs are not directly comparable to Cellcano's

## Evidence

- [other] Cellcano achieves the highest average accuracy (0.852), with scJoint second (0.837) and ACTINN third (0.782), confirming the values reported for Fig. 3a.: [evidence span withheld — anchor status: not_found]
- [discussion] We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods and integration with label transfer"
- [intro] we demonstrate that Cellcano is significantly more accurate, computationally efficient, and scalable compared to existing methods.: "we demonstrate that Cellcano is significantly more accurate, computationally efficient, and scalable compared to existing methods."
- [other] Load the Fig 3a sheet with pandas, extracting per-task accuracy values for the seven FACS-sorted gold-standard target tasks for each method.: [evidence span withheld — anchor status: not_found]
- [readme] Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications]: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications]"
