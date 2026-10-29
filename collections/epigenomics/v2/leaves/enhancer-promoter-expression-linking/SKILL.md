---
name: enhancer-promoter-expression-linking
description: Use when scHi-C-derived loop calls (e.g., SnapHiC *.postprocessed.summits.bedpe
  per cell type) must be connected to promoter-linked genes and tested whether anchors
  and linked genes show highest activity/expression in the matched cell type.
when_to_use_negative:
- When loops have not been called per cell type (no *.postprocessed.summits.bedpe
  outputs); SnapHiC loop calling must precede anchor/promoter quantification.
- When no cell type-matched epigenomic (ATAC/H3K27ac) or expression reference data
  exist for the cell types of interest, since matched-vs-unmatched enrichment cannot
  be computed.
- When the goal is bulk Hi-C loop detection from population contact maps rather than
  cell type-specific loops from size-matched single-cell datasets.
edam_operation: http://edamontology.org/operation_2437
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3169
tools:
- name: SnapHiC
  role: Identifies chromatin loops per cell type from scHi-C contact files; its summits
    output (*.postprocessed.summits.bedpe) supplies the anchors linked to promoters
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Quantifies CPM-normalized ATAC-seq/H3K27ac signal at loop anchors, assigns
    loops to linked promoters, and computes FPKM-based matched vs. unmatched enrichment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:d3f290f164efa1dc00f866dae5df4dbaca4435823ccdc180d116e73535ff251e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: http://edamontology.org/operation_2437
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# enhancer-promoter-expression-linking

## Summary

Links cell type-specific chromatin loops (identified with SnapHiC from matched-size scHi-C datasets) to their target promoters and validates the links with cell type-matched regulatory signatures: ATAC-seq accessibility, H3K27ac ChIP-seq signal, and FPKM gene expression.

## When to use

Use when scHi-C-derived loop calls (e.g., SnapHiC *.postprocessed.summits.bedpe per cell type) must be connected to promoter-linked genes and tested whether anchors and linked genes show highest activity/expression in the matched cell type.

## When NOT to use

- When loops have not been called per cell type (no *.postprocessed.summits.bedpe outputs); SnapHiC loop calling must precede anchor/promoter quantification.
- When no cell type-matched epigenomic (ATAC/H3K27ac) or expression reference data exist for the cell types of interest, since matched-vs-unmatched enrichment cannot be computed.
- When the goal is bulk Hi-C loop detection from population contact maps rather than cell type-specific loops from size-matched single-cell datasets.

## Inputs

- Per-cell-type loop summit files from SnapHiC (*.postprocessed.summits.bedpe) with fdr_dist column
- CPM-normalized ATAC-seq signal tracks per brain cell type
- H3K27ac ChIP-seq signal tracks per brain cell type (Nott et al. epigenomic reference)
- RNA-seq expression reference with FPKM values per cell type (Zhang et al.)
- 10KB-binned filter regions file (mappability <=0.8 bins plus ENCODE blacklist overlaps) for hg19
- Per-cell contact files (tab-separated chromosome/position columns) for each cell type

## Outputs

- Enrichment summary table (e.g., fig2b_cell_type_loop_enrichment.tsv) of matched vs. unmatched cell type anchor ATAC/H3K27ac and linked-gene FPKM
- Cell type-specific loop sets with assigned linked promoters/genes

## How to apply

Run SnapHiC separately for each matched cell type (e.g., astrocyte, microglia, oligodendrocyte, L2/3 excitatory neuron with equal cell counts) using 'bin rwr' then 'hic interaction postprocess' with 10KB binsize, hg19, and filter regions excluding bins with mappability <=0.8 and ENCODE blacklist overlaps. Extract loop summits from each cell type's *.postprocessed.summits.bedpe, using the fdr_dist column (false discovery rate with respect to local background at the same 1D genomic distance) as the loop-strength measure. Quantify CPM-normalized ATAC-seq and H3K27ac ChIP-seq signal at the loop anchor regions in Python, assign each loop to its linked promoter, and quantify FPKM expression for the linked genes. Finally, compare anchor ATAC/H3K27ac signals and linked-promoter FPKM against cell type-matched reference epigenomic (Nott et al.) and expression (Zhang et al.) datasets, computing matched vs. unmatched cell type enrichment to confirm that loops show highest anchor activity and linked-gene expression in their matched cell type.

## Related tools

- **SnapHiC** (Identifies chromatin loops per cell type from scHi-C contact files; its summits output (*.postprocessed.summits.bedpe) supplies the anchors linked to promoters) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Quantifies CPM-normalized ATAC-seq/H3K27ac signal at loop anchors, assigns loops to linked promoters, and computes FPKM-based matched vs. unmatched enrichment)

## Examples

```
python snap.py --step postprocess --genome hg19 --binsize 10000 --filter_file ext/hg19_filter_regions.txt --prefix AST
```

## Evaluation signals

- Anchor regions of each cell type's loops show significantly higher ATAC-seq and H3K27ac signal in the matched cell type than in other cell types
- Genes whose promoters are linked to a cell type's loops show significantly higher FPKM expression in the matched cell type (Fig. 2b pattern)
- All four cell types use equal-sized datasets (e.g., 261 cells each) so loop counts are comparable across conditions
- Loop summits are ranked/filtered by the fdr_dist column (local-background FDR at matched 1D genomic distance)
- Filter regions correctly exclude 10KB bins with mappability <=0.8 and ENCODE blacklist overlaps

## Limitations

- SnapHiC currently accepts only mm10, hg19 and hg38 genomes, so other references require preprocessing.
- Default RWR computation requires solving a large matrix inverse (size of the binned chromosome) and demands large memory; a sliding_window approximation is available as an alternative.
- Validation depends on the availability of matched reference datasets (Nott et al. epigenomics, Zhang et al. expression); unmatched references weaken the enrichment test.
- No changelog found in the repository; no software version number for SnapHiC is stated in the methods beyond the README's version 0.2.0.

## Evidence

- [other] fdr_dist as loop strength measure: [evidence span withheld — anchor status: not_found]
- [results] matched regulatory signatures finding: [evidence span withheld — anchor status: not_found]
- [readme] mappability filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [methods] SnapHiC five steps: "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR)"
- [readme] loop summits output: "including *.postprocessed.summits.bedpe*, *.postprocessed.all_candidates.bedpe*, and *allChr.[hic/cool]*"
