---
name: single-cell-clustering-of-calling-cards-data
description: Use when analyzing single-cell Calling Cards qbed data (e.g., mouse cortex
  Brd4 scCC from GSE148448) where insertions must be assigned to cell types via Scanpy
  Leiden/Louvain clustering of matched scRNA-seq before differential peak analysis.
when_to_use_negative:
- Input is bulk Calling Cards data rather than single-cell qbed/insertions with matched
  scRNA-seq — cell-type clustering is not applicable.
- The expression data are already clustered/labeled cell types, so re-clustering with
  Scanpy is unnecessary before peak–gene pairing.
- The goal is ChIP-seq or motif-only analysis of TF binding without a single-cell
  dimension.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: pycallingcards
  role: Core module for reading qbed insertions, calling peaks (CCcaller), annotating
    peaks, building cell-by-peak AnnData, and pairing peaks with gene expression (cc.tl.pair_peak_gene_sc)
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Peak caller used via cc.pp.call_peaks to pinpoint TF binding sites from the
    single-cell insertions after clustering defines cell groups
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Performs normalization and Leiden/Louvain clustering of matched scRNA-seq
    to define cell types; Pycallingcards' structure follows Scanpy
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Mudata
  role: Combines the cell-by-peak AnnData and the gene-expression AnnData into one
    MuData object for joint analysis
- name: bedtools
  role: With pybedtools, supports cc.pp.annotation of called peaks to their nearest
    genes
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:2c2d9c9a06a9068cffd3f058ceb292a7744d21c892fef93bee8799a5043804fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - Mudata
  - Scanpy
  - bedtools
  - pycallingcards
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-clustering-of-calling-cards-data

## Summary

Clusters cells from single-cell Calling Cards (scCC) experiments using Scanpy on matched scRNA-seq data, so that Brd4 (or other TF) insertion counts can be grouped by cortical cell type and paired with cell-type-specific peaks via Pycallingcards.

## When to use

Use when analyzing single-cell Calling Cards qbed data (e.g., mouse cortex Brd4 scCC from GSE148448) where insertions must be assigned to cell types via Scanpy Leiden/Louvain clustering of matched scRNA-seq before differential peak analysis.

## When NOT to use

- Input is bulk Calling Cards data rather than single-cell qbed/insertions with matched scRNA-seq — cell-type clustering is not applicable.
- The expression data are already clustered/labeled cell types, so re-clustering with Scanpy is unnecessary before peak–gene pairing.
- The goal is ChIP-seq or motif-only analysis of TF binding without a single-cell dimension.

## Inputs

- single-cell Calling Cards qbed insertion file (e.g., mouse cortex Brd4 scCC, GEO GSE148448; 111,382 insertions)
- matched single-cell RNA-seq expression data for the same cells
- genome annotation for peak-to-gene assignment (used by bedtools/pybedtools)

## Outputs

- clustered/annotated cell-type labels for the scCC cells (Scanpy Leiden/Louvain groups)
- cell-by-peak AnnData object of Brd4 insertion counts per called peak (e.g., 902 peaks)
- combined MuData object integrating peak and gene-expression modalities
- cell-type-specific peak calls and peak–gene pairing table (e.g., Mef2c at chr13:83141353-83148478; E130114P18Rik at chr4:97575305-97588788)

## How to apply

Load the scCC qbed insertion data with pycallingcards.reading.read_qbed (e.g., 111,382 insertions across 35,950 cells in the mouse cortex Brd4 dataset) and load the matched scRNA-seq expression matrix. Normalize and cluster the expression data with Scanpy using standard preprocessing and Leiden/Louvain clustering to define cortical cell types; Pycallingcards' structure deliberately follows Scanpy so the two AnnData objects interoperate. Call peaks on the insertions with cc.pp.call_peaks (CCcaller), annotate peaks with cc.pp.annotation backed by bedtools/pybedtools, and convert insertions into a cell-by-peak AnnData with cc.pp.make_Anndata. Combine the peak AnnData with the clustered gene-expression AnnData into a MuData object, then run cc.tl.rank_peak_groups for differential peaks per cell type and cc.tl.pair_peak_gene_sc to link cell-type-specific binding to nearby gene expression.

## Related tools

- **pycallingcards** (Core module for reading qbed insertions, calling peaks (CCcaller), annotating peaks, building cell-by-peak AnnData, and pairing peaks with gene expression (cc.tl.pair_peak_gene_sc)) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Peak caller used via cc.pp.call_peaks to pinpoint TF binding sites from the single-cell insertions after clustering defines cell groups) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Performs normalization and Leiden/Louvain clustering of matched scRNA-seq to define cell types; Pycallingcards' structure follows Scanpy) — https://scanpy.readthedocs.io/en/stable/
- **Mudata** (Combines the cell-by-peak AnnData and the gene-expression AnnData into one MuData object for joint analysis)
- **bedtools** (With pybedtools, supports cc.pp.annotation of called peaks to their nearest genes) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed = cc.rd.read_qbed('GSE148448_brd4_cortex.qbed'); cc.pp.call_peaks(qbed, groupby='celltype'); cc.pp.make_Anndata(qbed, peaks); cc.tl.pair_peak_gene_sc(adata_peak, adata_rna)
```

## Evaluation signals

- Called peak set reproduces the reported ~902 Brd4 peaks from the mouse cortex scCC reanalysis.
- Cell count and insertion count are preserved after read_qbed (e.g., 111,382 insertions across 35,950 cells).
- Peak–gene pairing table recovers the reported cell-type-specific examples: Mef2c at chr13:83141353-83148478 (strongest binding in excitatory neurons) and E130114P18Rik at chr4:97575305-97588788 (astrocytes and neuroblasts).
- The MuData object contains both the peak AnnData and gene-expression AnnData modalities with shared cell barcodes.
- Clustered cell types recapitulate known cortical populations (e.g., excitatory neurons, astrocytes, neuroblasts) used in differential peak analysis.

## Limitations

- Requires matched single-cell RNA-seq from the same cells; clustering cell types cannot be done from insertions alone.
- bedtools/pybedtools must be installed for the annotation step; the workflow will not complete without them.
- Peak calling results depend on the CCcaller parameters matching the reported set (~902 peaks); deviations may change downstream pairings.

## Evidence

- [intro] Pycallingcards interacts with Scanpy: [evidence span withheld — anchor status: not_found]
- [intro] Preprocessing (pp) pipeline: "**Preprocessing (pp)** helps to preprocess data from qbed data to call peaks, make annotation, make Anndata object and filter peaks."
- [methods] Differential peaks and pairing: "pair analysis is completed to find out potential hypothesis for the relationship between transcription factor bindings and gene expression"
- [methods] bedtools dependency: [evidence span withheld — anchor status: not_found]
- [methods] MuData combination: "Two Anndata objects can combine into one Mudata object."
- [abstract] Novel cell-type-specific binding sites: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding and gene expression relationships."
- [methods] GEO accession for the dataset: "it can be downloaded from [GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE148448)."
