---
name: cross-dataset-reference-enrichment-analysis
description: Use when cell type-specific genomic features (loops, peaks, enhancers)
  from one dataset must be validated against independent cell type-matched epigenomic
  or transcriptomic reference datasets.
when_to_use_negative:
- When the input features are not cell type-resolved (e.g., bulk loops or peaks from
  mixed populations) — matched-vs-unmatched enrichment is undefined without cell type
  labels.
- When no independent cell type-matched reference dataset exists for the tissue of
  interest; the skill's core comparison cannot be performed.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SnapHiC
  role: Produces the cell type-specific loop summits (*.postprocessed.summits.bedpe)
    whose anchors and linked promoters are tested for matched-cell-type enrichment
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Used to quantify CPM-normalized ATAC-seq/H3K27ac signal at loop anchors and
    FPKM expression of linked genes, and to compute matched vs. unmatched enrichment
    against reference datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:089ab522380570a7cf59e001b87d54f202de6aaf558920981c3c6404b015ef82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-dataset-reference-enrichment-analysis

## Summary

Validates cell type-specific features (e.g., SnapHiC loop anchors and linked promoters) by quantifying CPM-normalized ATAC-seq/H3K27ac signal and FPKM expression and testing whether they are highest in the matched cell type of external reference datasets (Nott et al. epigenomics, Zhang et al. expression).

## When to use

Use when cell type-specific genomic features (loops, peaks, enhancers) from one dataset must be validated against independent cell type-matched epigenomic or transcriptomic reference datasets.

## When NOT to use

- When the input features are not cell type-resolved (e.g., bulk loops or peaks from mixed populations) — matched-vs-unmatched enrichment is undefined without cell type labels.
- When no independent cell type-matched reference dataset exists for the tissue of interest; the skill's core comparison cannot be performed.

## Inputs

- Cell type-specific loop summits in *.postprocessed.summits.bedpe format (with fdr_dist column) from SnapHiC runs on equal-sized (261-cell) astrocyte, microglia, oligodendrocyte, and L2/3 neuron datasets
- CPM-normalized ATAC-seq signal tracks per brain cell type
- H3K27ac ChIP-seq signal tracks per brain cell type
- FPKM gene expression table per cell type
- Nott et al. cell type epigenomic reference dataset
- Zhang et al. cell type expression reference dataset

## Outputs

- fig2b_cell_type_loop_enrichment.tsv — enrichment summary of anchor ATAC-seq/H3K27ac signal and linked-promoter FPKM, matched vs. unmatched cell types
- Per-loop anchor signal quantifications (CPM) and linked-gene expression values (FPKM) per cell type

## How to apply

Extract the feature coordinates of interest (e.g., loop summits from *.postprocessed.summits.bedpe, using the fdr_dist column as loop strength) for each cell type. Quantify CPM-normalized ATAC-seq and H3K27ac ChIP-seq signal at the feature regions (loop anchors) using Python, assign each loop to its linked promoter, and quantify FPKM expression of the linked genes. Compare these matched measurements against external reference datasets — Nott et al. brain cell type epigenomic data and Zhang et al. expression data — computing matched vs. unmatched cell type enrichment for each of the four cell types (astrocyte, microglia, oligodendrocyte, L2/3 neuron). The rationale is that features truly specific to a cell type should show significantly higher accessibility, H3K27ac, and linked-gene expression in the matched cell type than in the other three, as reported for SnapHiC loops in Fig. 2b.

## Related tools

- **SnapHiC** (Produces the cell type-specific loop summits (*.postprocessed.summits.bedpe) whose anchors and linked promoters are tested for matched-cell-type enrichment) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Used to quantify CPM-normalized ATAC-seq/H3K27ac signal at loop anchors and FPKM expression of linked genes, and to compute matched vs. unmatched enrichment against reference datasets)

## Examples

```
python quantify_loop_enrichment.py --summits astro.postprocessed.summits.bedpe --atac atac_cpm.bw --h3k27ac h3k27ac_cpm.bw --fpkm expression_fpkm.tsv --reference nott_zhang --out fig2b_cell_type_loop_enrichment.tsv
```

## Evaluation signals

- Loop anchors show significantly higher ATAC-seq signal in the matched cell type than in the other three cell types.
- Loop anchors show significantly higher H3K27ac ChIP-seq signal in the matched cell type than in unmatched cell types.
- Genes whose promoters are linked to the loops show significantly higher FPKM expression in the matched cell type.
- The enrichment summary (fig2b_cell_type_loop_enrichment.tsv) reproduces the expected pattern reported in Fig. 2b: highest anchor activity and linked-gene expression in the matched cell type.
- Loop strength ordering (fdr_dist from the summits file) is consistent with anchor enrichment magnitude across cell types.

## Limitations

- Reference datasets must cover the same cell types (astrocyte, microglia, oligodendrocyte, L2/3 excitatory neuron) and be comparable to the scHi-C-derived features; mismatched ontologies or brain regions weaken the enrichment test.
- Loop-to-promoter assignment assumes the loop links the regulatory element to its target promoter, which is only a putative target prediction from SnapHiC.
- Equal-sized (261-cell) datasets per cell type were used to control for sequencing depth; unbalanced cell numbers can confound cross-cell-type loop comparisons.

## Evidence

- [results] Anchors of cell type-specific loops showed significantly higher ATAC-seq and H3K27ac ChIP-seq signals in the matched cell type compared to other cell types: [evidence span withheld — anchor status: not_found]
- [methods] using the fdr_dist column (false discovery rate with respect to local background at the same 1D genomic distance) as the measure of loop strength: [evidence span withheld — anchor status: not_found]
- [methods] Quantify CPM-normalized ATAC-seq and H3K27ac signal at the loop anchor regions of each cell type's loops using Python: [evidence span withheld — anchor status: not_found]
- [methods] computing matched vs. unmatched cell type enrichment: [evidence span withheld — anchor status: not_found]
- [methods] identifying loop candidates based on both global and local background models: "identifying loop candidates based on both global and local background models, and (5) postprocessing: clustering nearby loop candidates and identifying loop summits"
