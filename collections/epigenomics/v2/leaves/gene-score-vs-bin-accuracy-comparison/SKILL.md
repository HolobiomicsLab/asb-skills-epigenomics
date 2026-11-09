---
name: gene-score-vs-bin-accuracy-comparison
description: Use when evaluating which scATAC-seq input representation (ArchR-derived
  gene scores vs. 500-bp fixed-size bin counts) yields better Cellcano celltype prediction
  accuracy across benchmark tasks, especially mouse brain reference-target pairs from
  Fig 2.
when_to_use_negative:
- The goal is comparing Cellcano against other celltyping methods (Seurat, Signac,
  scJoint, EpiAnno) rather than comparing Cellcano's own input representations
- The tasks are human PBMC tasks or other non-mouse-brain tasks for which the 63-task
  / 62-win validation result does not apply
- Raw scATAC-seq data (fragment or bam files) needs processing first — run the ArchR
  pipeline to obtain gene scores before this accuracy comparison is meaningful
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cellcano
  role: Supervised celltyping method whose prediction accuracies under gene-score
    input and 500-bp bin input are being compared across mouse brain tasks
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Pipeline used by Cellcano to process raw scATAC-seq data and obtain the gene
    scores used as the alternative input representation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:e560d830461701d72a5f6ebb3085eabefb551cb54ceb8c65db67b249c29f277b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-score-vs-bin-accuracy-comparison

## Summary

A benchmark-comparison skill for Cellcano scATAC-seq celltyping that tabulates prediction accuracy using gene scores as input versus genome-wide fixed-size 500-bp bin counts, based on the Figure 2a-b Source Data of the Nature Communications Cellcano paper. It quantifies how often gene scores outperform bins across the 63 mouse brain celltyping tasks.

## When to use

Use when evaluating which scATAC-seq input representation (ArchR-derived gene scores vs. 500-bp fixed-size bin counts) yields better Cellcano celltype prediction accuracy across benchmark tasks, especially mouse brain reference-target pairs from Fig 2.

## When NOT to use

- The goal is comparing Cellcano against other celltyping methods (Seurat, Signac, scJoint, EpiAnno) rather than comparing Cellcano's own input representations
- The tasks are human PBMC tasks or other non-mouse-brain tasks for which the 63-task / 62-win validation result does not apply
- Raw scATAC-seq data (fragment or bam files) needs processing first — run the ArchR pipeline to obtain gene scores before this accuracy comparison is meaningful

## Inputs

- Source Data spreadsheet (Figure 2a-b) from the Nature Communications Cellcano article, DOI 10.1038/s41467-023-37439-3
- Per-task prediction accuracy values for Cellcano run with gene-score input on 63 mouse brain celltyping tasks
- Per-task prediction accuracy values for Cellcano run with 500-bp fixed-size bin count input on the same 63 mouse brain tasks

## Outputs

- fig2ab_comparison.csv containing 63 mouse brain task rows with side-by-side accuracies for gene scores vs. 500-bp bins plus a per-task win/loss flag
- Win count for gene scores vs. bins (expected: 62 out of 63)

## How to apply

Download the Source Data spreadsheet for Figure 2a-b from the Cellcano article (DOI 10.1038/s41467-023-37439-3) and load the relevant worksheets with pandas. Extract the prediction accuracy values for the gene-score input and the 500-bp fixed-size bin input across the 63 mouse brain celltyping tasks, tabulating per-task accuracies side by side. Compare the two columns per task with a pandas/numpy boolean comparison to flag wins for gene scores, then count the wins. The article's reported outcome is that gene scores beat 500-bp bins in 62 of 63 mouse brain prediction results, consistent with the paper's discussion that 'using gene scores works as equally well or even better than using bin counts as input.' Write the reproduced comparison table (per-task accuracies plus a win/loss flag) to fig2ab_comparison.csv.

## Related tools

- **Cellcano** (Supervised celltyping method whose prediction accuracies under gene-score input and 500-bp bin input are being compared across mouse brain tasks) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Pipeline used by Cellcano to process raw scATAC-seq data and obtain the gene scores used as the alternative input representation)

## Examples

```
import pandas as pd; df = pd.read_excel('Source_Data_Fig2ab.xlsx', sheet_name='Fig2a_b'); df['gene_score_wins'] = df['accuracy_gene_scores'] > df['accuracy_500bp_bins']; print(int(df['gene_score_wins'].sum()))  # expect 62 of 63
```

## Evaluation signals

- The reproduced comparison table (fig2ab_comparison.csv) contains exactly 63 mouse brain task rows
- The win count for gene scores vs. 500-bp bins equals 62 out of 63, matching the reported Figure 2a-b outcome
- Per-task accuracy values agree with the values in the downloaded Source Data spreadsheet for Figure 2a-b
- The win/loss flag is consistent with the per-task accuracy comparison (flag = gene_score_accuracy > bins_500bp_accuracy)
- The qualitative conclusion is consistent with the article's EnrichedIndex finding that gene scores work as equally well or better than bin counts

## Limitations

- The 62/63 result is specific to mouse brain celltyping tasks; the win count must be re-derived for other systems such as human PBMCs
- Only two-round Cellcano prediction accuracies are compared; the comparison does not transfer to other celltyping methods
- Cellcano has not been tested on Apple M1 hardware, which may affect environment setup for reproducing the underlying predictions

## Evidence

- [discussion] gene scores outperform bins across mouse brain tasks: "We show that using gene scores works as equally well or even better than using bin counts as input."
- [intro] ArchR produces the gene-score input: "Cellcano incorporates ArchR30 pipeline to process the raw data and obtain gene scores for both reference and target datasets"
- [readme] Cellcano scope and tool identity: "Cellcano is an open source software for supervised cell type identification (celltyping) in scATAC-seq data"
- [discussion] benchmark setting of the comparison: "We have shown in 50 celltyping tasks with data from two systems (human PBMCs and mouse brain) that Cellcano significantly outperforms other celltyping methods"
