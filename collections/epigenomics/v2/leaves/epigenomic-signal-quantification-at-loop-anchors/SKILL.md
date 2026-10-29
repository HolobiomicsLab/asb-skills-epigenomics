---
name: epigenomic-signal-quantification-at-loop-anchors
description: Use when you have cell type-specific chromatin loops (e.g. SnapHiC *.postprocessed.summits.bedpe)
  plus matched ATAC-seq, H3K27ac and RNA-seq reference data and need to validate loop
  anchors and linked genes against matched cell type signatures.
when_to_use_negative:
- Loops were not called per cell type (e.g. pooled bulk Hi-C loops), so matched vs.
  unmatched enrichment is not meaningful
- No cell type-matched ATAC-seq/H3K27ac/RNA-seq reference data exist for the cell
  types of interest
- The goal is loop calling itself rather than downstream validation — use the SnapHiC
  pipeline steps (bin, rwr, hic, interaction, postprocess) instead
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0204
tools:
- name: SnapHiC
  role: Upstream tool whose postprocessed loop summits (*.postprocessed.summits.bedpe)
    provide the anchor regions quantified here
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Used to quantify CPM-normalized ATAC-seq and H3K27ac signal at loop anchor
    regions and compute enrichment summaries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:d55f56e8067fe3d7f84642886e94c1850e66aa95b49b6d903f03e19fa80f9174
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Epigenomic signal quantification at loop anchors

## Summary

Quantifies CPM-normalized ATAC-seq and H3K27ac ChIP-seq signal at chromatin loop anchor regions and FPKM expression of loop-linked promoters, then tests whether cell type-specific loops (called with SnapHiC) are supported by cell type-matched regulatory signatures.

## When to use

Use when you have cell type-specific chromatin loops (e.g. SnapHiC *.postprocessed.summits.bedpe) plus matched ATAC-seq, H3K27ac and RNA-seq reference data and need to validate loop anchors and linked genes against matched cell type signatures.

## When NOT to use

- Loops were not called per cell type (e.g. pooled bulk Hi-C loops), so matched vs. unmatched enrichment is not meaningful
- No cell type-matched ATAC-seq/H3K27ac/RNA-seq reference data exist for the cell types of interest
- The goal is loop calling itself rather than downstream validation — use the SnapHiC pipeline steps (bin, rwr, hic, interaction, postprocess) instead

## Inputs

- SnapHiC loop summit files (*.postprocessed.summits.bedpe) per cell type with fdr_dist column
- CPM-normalized ATAC-seq signal tracks per brain cell type
- H3K27ac ChIP-seq signal tracks per brain cell type
- FPKM gene expression table per cell type
- Nott et al. epigenomic reference dataset
- Zhang et al. expression reference dataset

## Outputs

- Per-loop anchor ATAC-seq and H3K27ac CPM signal quantifications
- Loop-to-promoter assignments with linked-gene FPKM values
- Matched vs. unmatched cell type enrichment summary (fig2b_cell_type_loop_enrichment.tsv)

## How to apply

Extract loop summits from each cell type's *.postprocessed.summits.bedpe output, using the fdr_dist column (false discovery rate with respect to local background at the same 1D genomic distance) as the measure of loop strength. Quantify CPM-normalized ATAC-seq and H3K27ac signal at the loop anchor regions of each cell type's loops using Python. Assign each loop to its linked promoter and quantify FPKM gene expression for the linked genes. Compare anchor ATAC-seq/H3K27ac signals and linked-promoter FPKM expression against reference epigenomic (Nott et al.) and expression (Zhang et al.) datasets, computing matched vs. unmatched cell type enrichment. The rationale is that true cell type-specific loops should show highest anchor accessibility, H3K27ac, and linked-gene expression in their matched cell type (Fig. 2b).

## Related tools

- **SnapHiC** (Upstream tool whose postprocessed loop summits (*.postprocessed.summits.bedpe) provide the anchor regions quantified here) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Used to quantify CPM-normalized ATAC-seq and H3K27ac signal at loop anchor regions and compute enrichment summaries)

## Examples

```
python snap.py --step postprocess ... ; # then in Python: quantify CPM ATAC-seq/H3K27ac at summits anchors from *.postprocessed.summits.bedpe and write fig2b_cell_type_loop_enrichment.tsv
```

## Evaluation signals

- Anchors of cell type-specific loops show significantly higher ATAC-seq signal in the matched cell type than in other cell types
- Anchors of cell type-specific loops show significantly higher H3K27ac ChIP-seq signal in the matched cell type than in other cell types
- Genes whose promoters are linked to the loops show significantly higher FPKM expression in the matched cell type
- The enrichment summary TSV reproduces the Fig. 2b pattern (highest anchor activity and linked-gene expression in matched cell type)
- Loop strength ordering is consistent when ranked by the fdr_dist (local-background FDR) column

## Limitations

- Cell type specificity of the enrichment conclusion depends on comparing equal-sized datasets (261 cells per type: astrocyte, microglia, oligodendrocyte, L2/3 neuron); unequal sampling could confound loop counts
- SnapHiC removes 10KB bins with mappability <=0.8 and ENCODE blacklist overlaps upstream, so anchors in poorly mappable regions are absent by construction
- Promoter linking assigns each loop to a linked promoter; genes whose regulatory targets are not near a promoter of the loop anchor will not be captured by FPKM-based validation

## Evidence

- [other] Extract the loop summits from each cell type's *.postprocessed.summits.bedpe output, using the fdr_dist column (false discovery rate with respect to local background at the same 1D genomic distance) as the measure of loop strength.: [evidence span withheld — anchor status: not_found]
- [results] Anchors of cell type-specific loops showed significantly higher ATAC-seq and H3K27ac ChIP-seq signals in the matched cell type compared to other cell types, and genes whose promoters were linked to these loops showed significantly higher expression in the matched cell type.: [evidence span withheld — anchor status: not_found]
- [methods] We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions.: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] A binned bed file of the genomic regions that are excluded from loop calling, such as the low mappability regions and the ENCODE blacklist regions.: "A binned bed file of the genomic regions that are excluded from loop calling, such as the low mappability regions and the ENCODE blacklist regions."
