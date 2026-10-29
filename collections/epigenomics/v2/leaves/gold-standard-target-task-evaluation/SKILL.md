---
name: gold-standard-target-task-evaluation
description: Use when comparing supervised scATAC-seq celltyping methods on tasks
  whose target dataset carries FACS-sorted gold-standard cell labels (human PBMCs),
  or when validating reported benchmark accuracies against the paper's Source Data.
when_to_use_negative:
- Target data lacks FACS-sorted gold-standard labels (e.g., unlabeled target cells
  or non-PBMC datasets) — this skill reproduces the specific gold-standard benchmark,
  not the full 50-task panel
- The goal is to run Cellcano celltyping prediction on new data (gene scores via ArchR,
  F-test feature selection, two-round MLP/KD model) rather than to evaluate or reproduce
  published benchmark accuracies
- Input is already an aggregated per-method summary rather than the per-task accuracy
  table from the Source Data sheet
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Cellcano
  role: Supervised scATAC-seq celltyping method being benchmarked; achieves the highest
    average accuracy (0.852) on the seven gold-standard FACS-sorted PBMC target tasks
  repo: https://marvinquiet.github.io/Cellcano/
- name: scJoint
  role: Label-transfer comparison method; second in the gold-standard target-task
    ranking with average accuracy 0.837
- name: EpiAnno
  role: Supervised scCAS celltyping method contextualized as an alternative approach
    in the same evaluation landscape
  repo: https://github.com/xy-chen16/EpiAnno
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:92bf76c018d4daf0ffdcdb9160ca90361caa98a7c8b95fd42f81ad6f2b51232c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellcano
  - EpiAnno
  - scJoint
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gold-standard-target-task-evaluation

## Summary

Benchmarks supervised scATAC-seq celltyping methods (Cellcano, scJoint, ACTINN) on the seven celltyping tasks that use FACS-sorted gold-standard human PBMCs as target data, reproducing the average-accuracy ranking reported for Fig. 3a of the Cellcano Nature Communications paper.

## When to use

Use when comparing supervised scATAC-seq celltyping methods on tasks whose target dataset carries FACS-sorted gold-standard cell labels (human PBMCs), or when validating reported benchmark accuracies against the paper's Source Data.

## When NOT to use

- Target data lacks FACS-sorted gold-standard labels (e.g., unlabeled target cells or non-PBMC datasets) — this skill reproduces the specific gold-standard benchmark, not the full 50-task panel
- The goal is to run Cellcano celltyping prediction on new data (gene scores via ArchR, F-test feature selection, two-round MLP/KD model) rather than to evaluate or reproduce published benchmark accuracies
- Input is already an aggregated per-method summary rather than the per-task accuracy table from the Source Data sheet

## Inputs

- Source Data supplementary spreadsheet (.xlsx) of Nature Communications article 10.1038/s41467-023-37439-3, Fig 3a sheet
- Per-task prediction-accuracy values for the seven FACS-sorted gold-standard (human PBMC target) celltyping tasks
- Task metadata distinguishing gold-standard FACS-sorted target tasks from the other benchmark tasks (the paper benchmarks 50 tasks total)

## Outputs

- fig3a_accuracy_summary.csv — per-method average accuracy across the seven gold-standard target tasks (Cellcano, scJoint, ACTINN)
- Deviation flags for any method whose recomputed mean differs from the reported value (0.852 / 0.837 / 0.782) by more than 0.005

## How to apply

Download the Source Data supplementary file for article 10.1038/s41467-023-37439-3 and locate the sheet corresponding to Fig 3a. Load that sheet with pandas and extract per-task prediction-accuracy values for the seven FACS-sorted gold-standard target tasks, restricted to those tasks only. Group the rows by method and compute the mean accuracy per method — the expected ranking is Cellcano 0.852, scJoint 0.837, ACTINN 0.782. Flag any method whose recomputed mean deviates from the reported value by more than 0.005. Write the three-method summary of average accuracies to fig3a_accuracy_summary.csv.

## Related tools

- **Cellcano** (Supervised scATAC-seq celltyping method being benchmarked; achieves the highest average accuracy (0.852) on the seven gold-standard FACS-sorted PBMC target tasks) — https://marvinquiet.github.io/Cellcano/
- **scJoint** (Label-transfer comparison method; second in the gold-standard target-task ranking with average accuracy 0.837)
- **EpiAnno** (Supervised scCAS celltyping method contextualized as an alternative approach in the same evaluation landscape) — https://github.com/xy-chen16/EpiAnno

## Examples

```
import pandas as pd; df = pd.read_excel('SourceData.xlsx', sheet_name='Fig3a'); means = df[df.task.isin(gold_standard_tasks)].groupby('method').accuracy.mean(); means.round(3).to_csv('fig3a_accuracy_summary.csv')
```

## Evaluation signals

- Recomputed mean accuracy for Cellcano over the seven gold-standard target tasks equals 0.852 within the 0.005 tolerance
- Recomputed mean accuracies for scJoint (0.837) and ACTINN (0.782) match the reported Fig 3a values within 0.005
- Exactly seven FACS-sorted gold-standard target tasks are aggregated per method — no extra tasks from the 50-task benchmark leak into the average
- fig3a_accuracy_summary.csv exists and contains a three-method table with the expected descending ranking Cellcano > scJoint > ACTINN
- The values agree with what the paper reports for Fig. 3a, confirming the reproduction

## Limitations

- No specific software versions are reported for the compared tools (Seurat, Signac, scJoint) in the paper's discussion, so exact re-running of the comparison is under-specified
- The evaluation depends on the published Source Data sheet; per-task raw accuracy values are not independently re-derivable without re-running each method's prediction pipeline
- The ranking covers only scATAC-seq-referenced supervised celltyping on the gold-standard PBMC tasks; Cellcano's planned extensions (cross-modality scRNA-seq reference, scDNA-methylation celltyping, multimodal reference) are not implemented and not covered

## Evidence

- [abstract] systematic benchmark on 50 gold-standard tasks: "After systematically benchmarking Cellcano on 50 well-designed celltyping tasks from various datasets"
- [discussion] Cellcano outperforms other methods across PBMC and mouse brain tasks: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods and integration with label transfer"
- [readme] Cellcano is supervised scATAC-seq celltyping software: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data"
- [intro] Seurat and scJoint use scRNA-seq references for label transfer: "Seurat24 and scJoint25 use scRNA-seq as reference to transfer cell labels to scATAC-seq"
