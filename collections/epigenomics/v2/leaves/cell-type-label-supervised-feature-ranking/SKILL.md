---
name: cell-type-label-supervised-feature-ranking
description: 'Use when building a Cellcano reference for scATAC-seq celltyping: the
  reference gene score matrix (from ArchR) has annotated cell type labels and features
  must be reduced to cell-type-specific genes before MLP training.'
when_to_use_negative:
- Input is raw scATAC-seq data (fragment file or bam file) — gene scores must first
  be generated via the ArchR pipeline before F-test selection is possible
- Reference data lacks annotated cell type labels — the F-test is supervised and requires
  cell type labels to score genes
- Features are peaks or bin counts rather than gene scores — this selection step is
  defined on the reference gene score matrix
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cellcano
  role: Implements the F-test feature-selection step that ranks reference gene scores
    and selects cell-type-specific genes as model features
  repo: https://marvinquiet.github.io/Cellcano/
- name: ArchR
  role: Produces the reference gene score matrix that the F-test selection is applied
    to
  repo: https://www.archrproject.com/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-37439-3
    title: 'Cellcano: supervised cell type identification for single cell ATAC-seq
      data'
  source_content_hash: sha256:ba7fa36dd645413eb73503f272995d14e8fbadd34780cb763d2364aa60fe7aaf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ArchR
  - Cellcano
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-type-label-supervised-feature-ranking

## Summary

Selects cell-type-specific gene features for Cellcano's supervised scATAC-seq celltyping by ranking reference gene scores with an F-test and retaining the top 3000 genes as model input.

## When to use

Use when building a Cellcano reference for scATAC-seq celltyping: the reference gene score matrix (from ArchR) has annotated cell type labels and features must be reduced to cell-type-specific genes before MLP training.

## When NOT to use

- Input is raw scATAC-seq data (fragment file or bam file) — gene scores must first be generated via the ArchR pipeline before F-test selection is possible
- Reference data lacks annotated cell type labels — the F-test is supervised and requires cell type labels to score genes
- Features are peaks or bin counts rather than gene scores — this selection step is defined on the reference gene score matrix

## Inputs

- Reference scATAC-seq gene score matrix (cells x genes), obtained via the ArchR pipeline
- Reference metadata file containing known cell type labels for each reference cell

## Outputs

- selected_features.csv with exactly 3000 gene rows and a gene identifier field
- Gene list ranked in descending order of F-statistic

## How to apply

Load the reference scATAC-seq gene score matrix (produced by the ArchR pipeline) together with its metadata file of known cell type labels. Apply an F-test to each gene, testing for differences in gene score across the annotated cell type labels — this supervised selection keeps genes whose signal discriminates cell types, since Cellcano 'applies F-test on reference gene scores to select cell-type-specific genes as features for model construction'. Rank all genes by F-statistic in descending order and retain the top 3000 genes as the feature set. Write the selected genes to 'selected_features.csv' including the required gene identifier field. The gene score input (rather than bin counts) is justified by the authors' finding that 'using gene scores works as equally well or even better than using bin counts as input', and the resulting 3000 features feed the first-round MLP trained on reference gene scores.

## Related tools

- **Cellcano** (Implements the F-test feature-selection step that ranks reference gene scores and selects cell-type-specific genes as model features) — https://marvinquiet.github.io/Cellcano/
- **ArchR** (Produces the reference gene score matrix that the F-test selection is applied to) — https://www.archrproject.com/

## Examples

```
from scipy.stats import f_oneway; F,_ = f_oneway(*[X[y==c] for c in set(y)], axis=0); top = X.columns[np.argsort(-F)[:3000]]; top.to_frame().to_csv('selected_features.csv')
```

## Evaluation signals

- 'selected_features.csv' exists and contains exactly 3000 gene rows
- The file includes the required gene identifier field consistent with gene names in the reference gene score matrix
- An F-statistic was computed for every gene in the reference matrix across all annotated cell type labels, and the retained genes are the top-ranked by descending F-statistic
- The selected 3000 genes are the exact feature dimension used to construct the first-round MLP trained on reference gene scores

## Limitations

- Cellcano has not been tested on M1 Macs (per README, Issue #6 covers installation help)
- The 3000-gene cutoff is the tutorial/workflow threshold; it is a fixed top-N selection rather than a significance-based filter
- Feature selection requires the reference scATAC-seq data with cell type labels; adaptation to cross-modality references (e.g., scRNA-seq) or multimodal references is planned but not implemented
- GPU support is optional (CPU is sufficient), and rpy2/R plus ArchR are required only if starting from raw fragment or bam files

## Evidence

- [intro] Cellcano applies F-test on reference gene scores to select cell-type-specific genes as features for model construction31: "Cellcano applies F-test on reference gene scores to select cell-type-specific genes as features for model construction31"
- [discussion] We show that using gene scores works as equally well or even better than using bin counts as input.: "We show that using gene scores works as equally well or even better than using bin counts as input."
- [readme] If the input is scATAC-seq raw data (i.e. fragment file or bam file), [ArchR](https://www.archrproject.com/) package has to be installed.: [evidence span withheld — anchor status: external_doc]
- [intro] In the first round, Cellcano trains an MLP model with reference gene scores and predicts cell types in target data.: "In the first round, Cellcano trains an MLP model with reference gene scores and predicts cell types in target data."
