---
name: scrnaseq-count-matrix-normalization
description: Use when a raw or unnormalized scRNA-seq count matrix from two biological
  conditions (e.g. treated vs untreated cells) must be normalized in Seurat before
  calling differentially expressed genes for downstream regulatory-element queries.
when_to_use_negative:
- Input is bulk RNA-seq or already-normalized expression data where Seurat's FindAllMarkers
  per-cell ranking does not apply.
- The goal is REM retrieval directly from genomic regions rather than from a gene
  list — use EpiRegio's Region Query workflow (ChIP-seq BED overlap) instead of a
  Gene Query on DE genes.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Normalizes the scRNA-seq expression matrix and runs FindAllMarkers (Wilcoxon
    Rank Sum) to identify DE genes at p ≤ 0.01.
- name: EpiRegio
  role: Downstream consumer of the exported GeneSet.txt DE gene list via the Gene
    Query (REST API https://epiregio.de/REST_API/GeneQuery/<input>/) to retrieve associated
    REMs.
  repo: github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:ca2df88ad3a25cdc3d4199d857b75ce533cdd7f4ba362c52c32eccce3253e588
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scRNA-seq count matrix normalization

## Summary

Normalizes a single-cell RNA-seq expression matrix in Seurat so that differential expression between conditions (e.g. TGF-β-treated vs untreated HUVECs undergoing EndoMT) can be computed with FindAllMarkers using the Wilcoxon Rank Sum test at p ≤ 0.01, yielding a DE gene list for downstream REM retrieval via EpiRegio's Gene Query.

## When to use

Use when a raw or unnormalized scRNA-seq count matrix from two biological conditions (e.g. treated vs untreated cells) must be normalized in Seurat before calling differentially expressed genes for downstream regulatory-element queries.

## When NOT to use

- Input is bulk RNA-seq or already-normalized expression data where Seurat's FindAllMarkers per-cell ranking does not apply.
- The goal is REM retrieval directly from genomic regions rather than from a gene list — use EpiRegio's Region Query workflow (ChIP-seq BED overlap) instead of a Gene Query on DE genes.

## Inputs

- single-cell RNA-seq count/expression matrix (e.g. Glaser et al. HUVEC EndoMT dataset, doi.org/10.1073/pnas.1913481117)
- condition labels distinguishing TGF-β-treated versus untreated HUVECs

## Outputs

- normalized Seurat expression matrix
- list of differentially expressed genes (Wilcoxon Rank Sum, p ≤ 0.01), exported as GeneSet.txt

## How to apply

Load the scRNA-seq dataset (e.g. the Glaser et al. doi.org/10.1073/pnas.1913481117 dataset of TGF-β-treated versus untreated HUVECs undergoing endothelial-to-mesenchymal transition) into Seurat and normalize the expression matrix. With normalized values, run Seurat's FindAllMarkers using the Wilcoxon Rank Sum test to compare conditions, retaining genes with p ≤ 0.01. Export the significant DE gene list as GeneSet.txt, matching the format of the example file in the ApplicationScenarioExamples/identifyEnrichedTFs repository folder. The rationale is that normalization makes expression values comparable across cells so the rank-sum test identifies condition-specific markers, which are then submitted to EpiRegio's Gene Query (filtered for 'heart' cell type/tissue) to retrieve associated REMs.

## Related tools

- **Seurat** (Normalizes the scRNA-seq expression matrix and runs FindAllMarkers (Wilcoxon Rank Sum) to identify DE genes at p ≤ 0.01.)
- **EpiRegio** (Downstream consumer of the exported GeneSet.txt DE gene list via the Gene Query (REST API https://epiregio.de/REST_API/GeneQuery/<input>/) to retrieve associated REMs.) — github.com/TeamRegio/EpiRegioDB

## Examples

```
seurat_obj <- NormalizeData(seurat_obj, normalization.method = "LogNormalize"); markers <- FindAllMarkers(seurat_obj, test.use = "wilcox", only.pos = FALSE)
```

## Evaluation signals

- Differential expression step must report exactly 304 significant DE genes (metric_de_genes under cond_tgfb_huvec_endomt) when p ≤ 0.01 and the Wilcoxon Rank Sum test are used.
- Submitting the exported GeneSet.txt to EpiRegio's Gene Query with 'heart' in 'Filter for cell types/tissues' must return 11,836 REMs (metric_rems_de_genes under cond_tgfb_huvec_endomt).
- The exported Gene Query CSV must contain exactly 11,836 REM rows; a substantial shortfall indicates incorrect gene identifiers or filtering settings.
- GeneSet.txt format must match the example file in the ApplicationScenarioExamples/identifyEnrichedTFs repository folder.

## Limitations

- Validation depends on the 'heart' cell type/tissue filter in EpiRegio, which assumes regulatory processes in heart endothelial cells are comparable to the EndoMT context; other tissues may need different filters.
- The reported 304-gene / 11,836-REM metrics are specific to the Glaser et al. HUVEC dataset and the p ≤ 0.01 Wilcoxon threshold; other datasets or thresholds will not reproduce them.
- Incorrect gene identifiers (e.g. mixing gene symbols and Ensembl IDs) submitted to the Gene Query cause a substantial shortfall in retrieved REMs.

## Evidence

- [methods] Load the Glaser et al. (doi.org/10.1073/pnas.1913481117) single-cell RNA-seq dataset of TGF-beta-treated versus untreated Human Umbilical Endothelial Cells: [evidence span withheld — anchor status: not_found]
- [methods] Run Seurat's FindAllMarkers with the Wilcoxon Rank Sum test to identify differentially expressed genes between TGF-beta-treated and untreated HUVECs, retaining genes with p <= 0.01.: [evidence span withheld — anchor status: not_found]
- [results] 11,836 REMs were identified for the 304 genes differentially expressed in TGF-β-treated versus untreated HUVECs: [evidence span withheld — anchor status: not_found]
