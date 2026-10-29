---
name: single-cell-celltyping-accuracy-benchmarking
description: Use when validating or reproducing reported accuracy comparisons of scATAC-seq
  celltyping methods (e.g., Cellcano vs scJoint vs ACTINN) from a figure's Source
  Data file, specifically for tasks whose target is a FACS-sorted gold-standard dataset.
when_to_use_negative:
- You need to run the celltyping methods themselves on raw scATAC-seq data (e.g.,
  fragments/bam processed via ArchR gene scores) rather than reproduce published accuracy
  numbers from a Source Data table
- The benchmark tasks use a non-gold-standard target (not FACS-sorted human PBMCs)
  — this skill reproduces only the seven gold-standard target tasks of Fig 3a, not
  the full 50-task benchmark
- The comparison requires method runtime or scalability measurements rather than prediction
  accuracy
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Cellcano
  role: Top-ranked method whose reported average accuracy (0.852) on the seven gold-standard
    tasks is recomputed and verified
  repo: https://marvinquiet.github.io/Cellcano/
- name: scJoint
  role: Comparison method (scRNA-seq reference label transfer) whose Fig 3a average
    accuracy (0.837) is extracted and verified
- name: ACTINN
  role: Comparison method whose Fig 3a average accuracy (0.782) is extracted and verified
- name: pandas
  role: Used to load the Fig 3a Source Data sheet, group by method, and compute mean
    accuracies
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:a443ebb1e86b74b6cd2c013c2bf7e94b9314862cc33d65da05d8606776b4caed
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ACTINN
  - Cellcano
  - pandas
  - scJoint
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-celltyping-accuracy-benchmarking

## Summary

Reproduce and verify published accuracy rankings of supervised scATAC-seq celltyping methods by extracting per-task prediction accuracies from a paper's Source Data (e.g., the Fig 3a gold-standard FACS-sorted human PBMC tasks of the Cellcano paper) and recomputing per-method averages.

## When to use

Use when validating or reproducing reported accuracy comparisons of scATAC-seq celltyping methods (e.g., Cellcano vs scJoint vs ACTINN) from a figure's Source Data file, specifically for tasks whose target is a FACS-sorted gold-standard dataset.

## When NOT to use

- You need to run the celltyping methods themselves on raw scATAC-seq data (e.g., fragments/bam processed via ArchR gene scores) rather than reproduce published accuracy numbers from a Source Data table
- The benchmark tasks use a non-gold-standard target (not FACS-sorted human PBMCs) — this skill reproduces only the seven gold-standard target tasks of Fig 3a, not the full 50-task benchmark
- The comparison requires method runtime or scalability measurements rather than prediction accuracy

## Inputs

- Source Data supplementary Excel file for Nature Communications article 10.1038/s41467-023-37439-3, containing the Fig 3a sheet
- Per-task prediction accuracy values for the seven FACS-sorted gold-standard human PBMCs target celltyping tasks
- List of benchmarked methods to summarize (Cellcano, scJoint, ACTINN)

## Outputs

- fig3a_accuracy_summary.csv containing average accuracy per method (expected: Cellcano 0.852, scJoint 0.837, ACTINN 0.782)
- Flagged deviations list of any method whose recomputed mean accuracy differs from the reported value by more than 0.005

## How to apply

Download the Source Data supplementary file of the Nature Communications article 10.1038/s41467-023-37439-3 and locate the sheet corresponding to Fig 3a. Load that sheet with pandas and extract the per-task accuracy values for the seven celltyping tasks whose target is the FACS-sorted gold-standard human PBMCs dataset, keeping one accuracy value per (method, task) pair. Group by method and compute the mean accuracy for Cellcano, scJoint, and ACTINN across the seven tasks. Compare each recomputed mean against the reported values — Cellcano 0.852, scJoint 0.837, ACTINN 0.782 — and flag any method whose recomputed mean deviates by more than 0.005. Write the three-method summary table of average accuracies to fig3a_accuracy_summary.csv, with Cellcano expected to rank highest.

## Related tools

- **Cellcano** (Top-ranked method whose reported average accuracy (0.852) on the seven gold-standard tasks is recomputed and verified) — https://marvinquiet.github.io/Cellcano/
- **scJoint** (Comparison method (scRNA-seq reference label transfer) whose Fig 3a average accuracy (0.837) is extracted and verified)
- **ACTINN** (Comparison method whose Fig 3a average accuracy (0.782) is extracted and verified)
- **pandas** (Used to load the Fig 3a Source Data sheet, group by method, and compute mean accuracies)

## Examples

```
import pandas as pd; df = pd.read_excel('SourceData.xlsx', sheet_name='Fig3a'); means = df.groupby('method')['accuracy'].mean().round(3); means.to_csv('fig3a_accuracy_summary.csv')
```

## Evaluation signals

- Recomputed mean accuracies match reported values within the 0.005 tolerance: Cellcano 0.852, scJoint 0.837, ACTINN 0.782
- Exactly seven gold-standard (FACS-sorted target) tasks are aggregated per method — no other Fig 3a tasks leak into the mean
- fig3a_accuracy_summary.csv contains one row per benchmarked method with a mean accuracy column, and Cellcano ranks highest
- Zero methods are flagged after the deviation check (recomputed mean minus reported mean has absolute value <= 0.005 for all three)

## Limitations

- Only the seven FACS-sorted gold-standard human PBMC target tasks are covered; the Cellcano benchmark comprises 50 celltyping tasks from human PBMCs and mouse brain, so conclusions do not generalize to the full task set
- No specific versions are reported for the compared tools (e.g., Seurat, Signac, scJoint), so the reproduced ranking is version-agnostic and may not match re-runs with current tool versions
- Accuracy values are taken from the published Source Data sheet rather than re-derived from raw data; errors in the supplementary file propagate directly

## Evidence

- [intro] ArchR pipeline grounding for the benchmarked tasks: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [intro] Claim the accuracy ranking verifies: "we demonstrate that Cellcano is significantly more accurate, computationally efficient, and scalable compared to existing methods."
- [abstract] Scope of the benchmark being reproduced: "After systematically benchmarking Cellcano on 50 well-designed celltyping tasks from various datasets"
- [discussion] Cellcano vs competitor methods (Seurat, Signac, scJoint): "Cellcano does not need to jointly operate on the reference and target datasets, like Seurat, Signac, and scJoint does."
- [readme] Tool identity and provenance from README: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in Nature Communications"
