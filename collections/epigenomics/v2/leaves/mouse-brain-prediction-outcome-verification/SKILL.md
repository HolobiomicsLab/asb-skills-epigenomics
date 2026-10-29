---
name: mouse-brain-prediction-outcome-verification
description: Decision support for choosing between gene scores and 500-bp bin counts
  as scATAC-seq celltyping input, or when verifying the reported 62/63 mouse brain
  win count of the Cellcano benchmark before citing or extending it.
when_to_use_negative:
- You need to run Cellcano itself on raw scATAC-seq data (fragment/bam files) — this
  skill verifies published results, it does not execute the two-round supervised learning
  pipeline.
- You are reproducing method-comparison figures (Cellcano vs Seurat/Signac/scJoint/EpiAnno)
  or the human PBMC tasks rather than the gene-scores-vs-bins comparison on the 63
  mouse brain tasks.
- Your input is already a computed accuracy table or the Source Data worksheets are
  unavailable — there is no spreadsheet to parse and reproduce.
tools:
- name: Cellcano
  role: Supervised two-round celltyping method whose published Figure 2a-b mouse brain
    accuracies (gene scores vs 500-bp bins) are extracted and verified
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Pipeline incorporated by Cellcano to process raw scATAC-seq data and obtain
    the gene-score inputs whose performance is compared against 500-bp bin counts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:19661a603393e51f5d3458ba5138f70821bcb2fde65fb788b2884cc417c4347b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mouse-brain-prediction-outcome-verification

## Summary

Reproduces and verifies the Figure 2a-b outcome of the Cellcano paper (DOI 10.1038/s41467-023-37439-3) that gene scores outperform genome-wide fixed-size 500-bp bin counts as scATAC-seq celltyping input in 62 of 63 mouse brain prediction tasks, by tabulating per-task prediction accuracies from the Source Data spreadsheet.

## When to use

Decision support for choosing between gene scores and 500-bp bin counts as scATAC-seq celltyping input, or when verifying the reported 62/63 mouse brain win count of the Cellcano benchmark before citing or extending it.

## When NOT to use

- You need to run Cellcano itself on raw scATAC-seq data (fragment/bam files) — this skill verifies published results, it does not execute the two-round supervised learning pipeline.
- You are reproducing method-comparison figures (Cellcano vs Seurat/Signac/scJoint/EpiAnno) or the human PBMC tasks rather than the gene-scores-vs-bins comparison on the 63 mouse brain tasks.
- Your input is already a computed accuracy table or the Source Data worksheets are unavailable — there is no spreadsheet to parse and reproduce.

## Inputs

- Source Data spreadsheet for Figure 2a-b of the Cellcano Nature Communications article (DOI 10.1038/s41467-023-37439-3), downloaded from the journal site
- Worksheet columns containing per-task prediction accuracy values for the gene-score input condition and the 500-bp fixed-size bin count input condition

## Outputs

- fig2ab_comparison.csv: a 63-row table with per-task prediction accuracies for gene scores and 500-bp bins plus a per-task win/loss flag
- Gene-score win count over the 63 mouse brain tasks (expected value: 62)

## How to apply

Download the Source Data spreadsheet for Figure 2a-b from the Nature Communications article (DOI 10.1038/s41467-023-37439-3) and load the relevant worksheets with pandas. Extract prediction accuracy values for the two input conditions — gene scores (produced via the ArchR pipeline) and genome-wide fixed-size 500-bp bin counts — across the 63 mouse brain celltyping tasks. Tabulate per-task accuracies side by side and use pandas/numpy comparisons to count tasks where gene scores beat bins, adding a win/loss flag per task. Cross-check the resulting win count (expected: 62 of 63) against the article's discussion-level finding that gene scores work as equally well or even better than bin counts. Write the comparison table to fig2ab_comparison.csv, which must contain exactly 63 mouse brain task rows and a gene-score win count of 62 to be considered a successful reproduction.

## Related tools

- **Cellcano** (Supervised two-round celltyping method whose published Figure 2a-b mouse brain accuracies (gene scores vs 500-bp bins) are extracted and verified) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Pipeline incorporated by Cellcano to process raw scATAC-seq data and obtain the gene-score inputs whose performance is compared against 500-bp bin counts)

## Examples

```
import pandas as pd; sheets = pd.read_excel('SourceData_Fig2ab.xlsx', sheet_name=None); df = pd.concat([s.assign(condition=n) for n, s in sheets.items()]); wins = int((df['gene_score_accuracy'] > df['bins_500bp_accuracy']).sum()); print('gene-score wins:', wins, 'of', len(df))
```

## Evaluation signals

- The reproduced comparison table (fig2ab_comparison.csv) contains exactly 63 mouse brain task rows, one per celltyping task in Figure 2a-b.
- The count of tasks where gene-score accuracy exceeds 500-bp bin-count accuracy equals 62, matching the article's reported outcome.
- Per-task accuracy values extracted from the Source Data worksheets match the values plotted in Figure 2a-b of the paper.
- Each per-task win/loss flag is consistent with the sign of (gene-score accuracy minus 500-bp bin accuracy) computed with pandas/numpy comparisons.
- The 62/63 result is consistent with the article's qualitative claim that gene scores work as equally well or even better than bin counts as input.

## Limitations

- The 62/63 win count is specific to the mouse brain celltyping tasks in Figure 2a-b; it does not cover the human PBMC tasks or the method-vs-method comparisons elsewhere in the paper.
- Verification depends on the availability and exact worksheet layout of the journal's Source Data spreadsheet; a different layout requires re-mapping columns.
- The Cellcano package has not been tested on M1 Macs, per the repository README, which could hinder re-running predictions rather than only verifying the spreadsheet.

## Evidence

- [discussion] gene scores beat bins in mouse brain tasks: "We show that using gene scores works as equally well or even better than using bin counts as input."
- [intro] gene scores are obtained via ArchR: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [discussion] benchmark scope includes mouse brain tasks: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain)"
- [readme] Cellcano publication identity for Source Data download: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data, published in [Nature Communications](https://doi.org/10.1038/s41467-023-37439-3)."
