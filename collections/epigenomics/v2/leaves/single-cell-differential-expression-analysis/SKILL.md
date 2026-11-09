---
name: single-cell-differential-expression-analysis
description: Use when a normalized single-cell RNA-seq matrix contains two or more
  condition groups (e.g. TGF-β-treated vs untreated HUVECs undergoing EndoMT) and
  a per-gene significance list is needed as input for regulatory-element queries.
when_to_use_negative:
- Input is bulk RNA-seq or already-aggregated expression data rather than cell-level
  single-cell counts — Seurat's FindAllMarkers per-cell Wilcoxon framework does not
  apply
- The goal is to rank REMs or identify enriched TF motifs from pre-existing REM sequences
  (use EpiRegio Region Query plus PASTAA instead of DE analysis)
- Gene identifiers or Seurat identity (condition) assignments have not been established,
  since incorrect identifiers are a stated failure mode of the downstream Gene Query
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Normalizes the single-cell expression matrix and runs FindAllMarkers (Wilcoxon
    Rank Sum) to identify differentially expressed genes between TGF-β-treated and
    untreated HUVECs
- name: EpiRegio
  role: Consumes the exported DE gene list (GeneSet.txt) via the Gene Query (https://epiregio.de/geneQuery/
    or REST API) to retrieve associated REMs filtered by cell type/tissue
  repo: https://github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:83cca53dec922c65049c59b8740b7490c10f86b5c0dec0abafab36a0c1019960
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - Seurat
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-differential-expression-analysis

## Summary

Identifies genes differentially expressed between cell conditions in single-cell RNA-seq data using Seurat's FindAllMarkers with the Wilcoxon Rank Sum test, producing a filtered gene list (e.g. 304 genes at P ≤ 0.01 for TGF-β-treated vs untreated HUVECs) suitable for downstream REM retrieval via EpiRegio's Gene Query.

## When to use

Use when a normalized single-cell RNA-seq matrix contains two or more condition groups (e.g. TGF-β-treated vs untreated HUVECs undergoing EndoMT) and a per-gene significance list is needed as input for regulatory-element queries.

## When NOT to use

- Input is bulk RNA-seq or already-aggregated expression data rather than cell-level single-cell counts — Seurat's FindAllMarkers per-cell Wilcoxon framework does not apply
- The goal is to rank REMs or identify enriched TF motifs from pre-existing REM sequences (use EpiRegio Region Query plus PASTAA instead of DE analysis)
- Gene identifiers or Seurat identity (condition) assignments have not been established, since incorrect identifiers are a stated failure mode of the downstream Gene Query

## Inputs

- Single-cell RNA-seq expression matrix (e.g. Glaser et al. TGF-β-treated vs untreated HUVEC EndoMT dataset, doi.org/10.1073/pnas.1913481117)
- Cell-level condition/identity annotations distinguishing treated vs untreated cells

## Outputs

- Table of differentially expressed genes from FindAllMarkers (Wilcoxon Rank Sum) with p-values, filtered to P ≤ 0.01 (304 genes for cond_tgfb_huvec_endomt)
- GeneSet.txt gene list formatted per the ApplicationScenarioExamples/identifyEnrichedTFs example file

## How to apply

Load the single-cell RNA-seq dataset (e.g. Glaser et al., doi.org/10.1073/pnas.1913481117, TGF-β-treated vs untreated HUVECs undergoing endothelial-to-mesenchymal transition) and normalize the expression matrix in Seurat. Set the cell grouping so treated and untreated cells form distinct identities, then run Seurat's FindAllMarkers with the Wilcoxon Rank Sum test ('wilcox') to compute per-gene differential expression between conditions. Retain only genes passing the significance threshold — P ≤ 0.01 in the TGF-β/HUVEC scenario, which yielded 304 differentially expressed genes. Export the surviving gene list as GeneSet.txt, matching the format of the example file in the ApplicationScenarioExamples/identifyEnrichedTFs repository folder, so it can be submitted directly to EpiRegio's Gene Query. Validation hinges on the marker count: a large deviation from the expected 304 genes indicates incorrect normalization, identity assignment, or threshold settings.

## Related tools

- **Seurat** (Normalizes the single-cell expression matrix and runs FindAllMarkers (Wilcoxon Rank Sum) to identify differentially expressed genes between TGF-β-treated and untreated HUVECs)
- **EpiRegio** (Consumes the exported DE gene list (GeneSet.txt) via the Gene Query (https://epiregio.de/geneQuery/ or REST API) to retrieve associated REMs filtered by cell type/tissue) — https://github.com/TeamRegio/EpiRegioDB

## Examples

```
huvec <- NormalizeData(huvec); Idents(huvec) <- "condition"; de <- FindAllMarkers(huvec, test.use = "wilcox"); de304 <- subset(de, p_val <= 0.01); writeLines(unique(de304$gene), "GeneSet.txt")
```

## Evaluation signals

- The filtered marker table contains exactly 304 differentially expressed genes for the TGF-β-treated vs untreated HUVEC comparison (metric_de_genes under cond_tgfb_huvec_endomt)
- Submitting the exported GeneSet.txt to EpiRegio's Gene Query with 'heart' in 'Filter for cell types/tissues' returns 11,836 REMs (metric_rems_de_genes); a substantial shortfall indicates incorrect gene identifiers or filtering settings
- GeneSet.txt matches the format of the example file in the ApplicationScenarioExamples/identifyEnrichedTFs repository folder
- All retained markers satisfy P ≤ 0.01 under the Wilcoxon Rank Sum test
- The Seurat object is normalized and identities are assigned to treated/untreated groups before FindAllMarkers is invoked (otherwise marker counts are invalid)

## Limitations

- Wilcoxon Rank Sum marker calls at P ≤ 0.01 are specific to the Seurat pipeline; alternative tests or thresholds will not reproduce the 304-gene input expected by the EpiRegio Gene Query workflow
- The downstream EpiRegio retrieval relies on the 'heart' cell-type filter as a proxy, since endothelial cells within the heart undergo EndoMT and the regulatory processes are assumed comparable — an assumption, not a direct tissue match
- A substantial shortfall in retrieved REMs (below 11,836) signals incorrect gene identifiers or filtering settings in either the DE step or the Gene Query submission

## Evidence

- [methods] Run Seurat's FindAllMarkers with the Wilcoxon Rank Sum test to identify differentially expressed genes between TGF-beta-treated and untreated HUVECs, retaining genes with p <= 0.01.: [evidence span withheld — anchor status: not_found]
- [results] 11,836 REMs were identified for the 304 genes differentially expressed in TGF-β-treated versus untreated HUVECs: [evidence span withheld — anchor status: not_found]
- [methods] a substantial shortfall indicates incorrect gene identifiers or filtering settings: [evidence span withheld — anchor status: not_found]
- [readme] For application scenario "How to use EpiRegio to identify enriched TFs of a set of genes of interest" the examples and the workflow script can be found in the folder identifyEnrichedTFs.: [evidence span withheld — anchor status: external_doc]
