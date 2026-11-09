---
name: peak-gene-expression-pairing
description: Use when you have single-cell calling cards insertions (qbed) plus matched
  scRNA-seq for the same cells and need to identify TF binding peaks whose cell-type-specific
  binding correlates with expression of nearby genes.
when_to_use_negative:
- Input is already a peak-gene association table or a feature matrix without per-cell
  calling cards insertions
- The task is bulk calling cards peak-gene pairing rather than the single-cell (scCC)
  pairing workflow
- No matched gene-expression data exist for the calling cards cells, so expression
  correlation cannot be computed
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: pycallingcards
  role: Provides cc.tl.pair_peak_gene_sc, cc.tl.rank_peak_groups, cc.pp.make_Anndata,
    and the MuData integration used to pair peaks with gene expression
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Normalizes and clusters the matched scRNA-seq cells to define the cell types
    over which differential peaks and pairings are computed
- name: bedtools
  role: Backs cc.pp.annotation so each called peak is assigned its nearest genes for
    pairing
- name: Mudata
  role: Combines the cell-by-peak AnnData and the gene-expression AnnData into one
    object for joint pairing analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:0f16b0f9c458a173e07684a8110228f932fbe259e036f65ada2a24f246e919ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - Mudata
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# peak-gene-expression-pairing

## Summary

Pairs cell-type-specific calling cards TF binding peaks with nearby gene expression using Pycallingcards' cc.tl.pair_peak_gene_sc, after building a cell-by-peak AnnData and combining it with scRNA-seq data into a MuData object.

## When to use

Use when you have single-cell calling cards insertions (qbed) plus matched scRNA-seq for the same cells and need to identify TF binding peaks whose cell-type-specific binding correlates with expression of nearby genes.

## When NOT to use

- Input is already a peak-gene association table or a feature matrix without per-cell calling cards insertions
- The task is bulk calling cards peak-gene pairing rather than the single-cell (scCC) pairing workflow
- No matched gene-expression data exist for the calling cards cells, so expression correlation cannot be computed

## Inputs

- qbed insertion file for single-cell calling cards data (e.g., from GEO accession GSE148448)
- matched scRNA-seq expression AnnData clustered into cell types with Scanpy (Leiden/Louvain)
- peak annotations (nearest genes) from cc.pp.annotation using bedtools/pybedtools
- cell-by-peak AnnData from cc.pp.make_Anndata combined into a MuData object

## Outputs

- peak-gene pairing table from cc.tl.pair_peak_gene_sc linking cell-type-specific peaks to nearby gene expression
- differential peak rankings per cell type from cc.tl.rank_peak_groups
- putative binding-expression relationships (e.g., Mef2c and E130114P18Rik pairs)

## How to apply

Call peaks from qbed insertions with CCcaller via cc.pp.call_peaks (e.g., the Brd4 mouse cortex dataset GSE148448 yields ~902 peaks) and annotate each peak with its nearest genes using cc.pp.annotation backed by bedtools/pybedtools. Build a cell-by-peak AnnData with cc.pp.make_Anndata and combine it with the gene-expression AnnData into a single MuData object, as Pycallingcards states 'Two Anndata objects can combine into one Mudata object.' Run differential peak analysis per cell type with cc.tl.rank_peak_groups, then apply cc.tl.pair_peak_gene_sc to pair differential peaks with nearby gene expression and generate hypotheses about binding-expression relationships. Validate by recovering known cell-type-specific examples, e.g., peak chr13:83141353-83148478 pairing with Mef2c in excitatory neurons and chr4:97575305-97588788 pairing with E130114P18Rik in astrocytes and neuroblasts.

## Related tools

- **pycallingcards** (Provides cc.tl.pair_peak_gene_sc, cc.tl.rank_peak_groups, cc.pp.make_Anndata, and the MuData integration used to pair peaks with gene expression) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Normalizes and clusters the matched scRNA-seq cells to define the cell types over which differential peaks and pairings are computed)
- **bedtools** (Backs cc.pp.annotation so each called peak is assigned its nearest genes for pairing)
- **Mudata** (Combines the cell-by-peak AnnData and the gene-expression AnnData into one object for joint pairing analysis)

## Examples

```
import pycallingcards as cc; peaks = cc.pp.call_peaks(qbed); cc.pp.annotation(peaks); adata_peak = cc.pp.make_Anndata(qbed, peaks); mdata = mu.MuData({'peak': adata_peak, 'rna': adata_rna}); cc.tl.rank_peak_groups(mdata['peak'], 'leiden'); pairs = cc.tl.pair_peak_gene_sc(mdata)
```

## Evaluation signals

- Called peak set reproduces the reported ~902 peaks for the Brd4 mouse cortex reanalysis
- Pairing table recovers Mef2c paired with peak chr13:83141353-83148478 in excitatory neurons
- Pairing table recovers E130114P18Rik paired with peak chr4:97575305-97588788 in astrocytes and neuroblasts
- MuData object successfully contains both the cell-by-peak counts and gene-expression layers for all shared cells

## Limitations

- Pairing requires the qbed insertions and scRNA-seq to be from matched cells; discordant cell type labels between modalities will break the analysis
- Peak annotation with bedtools/pybedtools must be installed beforehand; annotation errors propagate into the pairing output
- The pairing output is described as producing 'potential hypothesis' — associations require downstream validation

## Evidence

- [methods] pair analysis is completed to find out potential hypothesis for the relationship between transcription factor bindings and gene expression: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [methods] Two Anndata objects combine into MuData for pairing: "Two Anndata objects can combine into one Mudata object."
- [readme] README scope of tools (tl) module: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [abstract] Novel cell-type-specific binding sites found by reanalysis: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding and gene expression relationships."
