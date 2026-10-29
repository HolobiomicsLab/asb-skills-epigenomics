---
name: cell-type-specific-loop-summit-annotation
description: Use when scHi-C-derived loops from matched cell type subsets (e.g. 261-cell
  astrocyte/microglia/ODC/L2/3 neuron SnapHiC runs) need epigenomic and expression
  validation against matched bulk references.
when_to_use_negative:
- When loops come from pooled (non-cell-type-resolved) Hi-C and no matched cell type
  epigenomic/expression references exist — matched vs. unmatched contrast cannot be
  computed.
- When the goal is loop discovery itself rather than biological validation of already-called
  summits — use the SnapHiC pipeline steps ('bin', 'rwr', 'interaction', 'postprocess')
  instead.
- When references use a different genome build than the loops (hg19 vs. mm10/hg38)
  without liftover — anchors will not align to the reference signal tracks.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0605
tools:
- name: SnapHiC
  role: Identifies cell type-specific chromatin loops (loop summits in *.postprocessed.summits.bedpe)
    from scHi-C contact files per cell type; its fdr_dist column supplies the loop
    strength measure used for enrichment testing
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Quantifies CPM-normalized ATAC-seq/H3K27ac at loop anchors and FPKM for linked
    promoters, and computes matched vs. unmatched enrichment statistics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:6b592fc8a0190499ffc90ee12061963d86e69ef4308e797ea97d00d47976b689
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cell-type-specific-loop-summit-annotation

## Summary

Annotates SnapHiC-identified cell type-specific chromatin loops by testing whether loop anchors carry cell type-matched ATAC-seq, H3K27ac ChIP-seq, and linked-gene RNA-seq enrichment (Nott et al. epigenomic and Zhang et al. expression references), validating loop calls biologically.

## When to use

Use when scHi-C-derived loops from matched cell type subsets (e.g. 261-cell astrocyte/microglia/ODC/L2/3 neuron SnapHiC runs) need epigenomic and expression validation against matched bulk references.

## When NOT to use

- When loops come from pooled (non-cell-type-resolved) Hi-C and no matched cell type epigenomic/expression references exist — matched vs. unmatched contrast cannot be computed.
- When the goal is loop discovery itself rather than biological validation of already-called summits — use the SnapHiC pipeline steps ('bin', 'rwr', 'interaction', 'postprocess') instead.
- When references use a different genome build than the loops (hg19 vs. mm10/hg38) without liftover — anchors will not align to the reference signal tracks.

## Inputs

- Per-cell-type SnapHiC postprocessed summit files (*.postprocessed.summits.bedpe) with fdr_dist column
- CPM-normalized ATAC-seq signal tracks per brain cell type (Nott et al.)
- CPM-normalized H3K27ac ChIP-seq signal tracks per brain cell type (Nott et al.)
- FPKM gene expression matrix per cell type (Zhang et al.)
- hg19 chrom.sizes and 10KB filter regions file (mappability <=0.8 + ENCODE blacklist)

## Outputs

- fig2b_cell_type_loop_enrichment.tsv: matched vs. unmatched cell type enrichment summary for anchor ATAC-seq, H3K27ac, and linked-gene FPKM
- Cell type-specific loop summit sets with linked promoters

## How to apply

Run SnapHiC (python snap.py, steps 'bin rwr' then 'hic interaction postprocess') separately for each equal-sized (~261-cell) cell type at 10KB binsize with hg19, filtering bins with mappability <=0.8 and ENCODE blacklist overlaps. Extract loop summits from each cell type's *.postprocessed.summits.bedpe, using the fdr_dist column (false discovery rate vs. local background at matched 1D genomic distance) as the loop strength measure. Quantify CPM-normalized ATAC-seq and H3K27ac signal at loop anchor regions in Python, assign each loop to its linked promoter, and compute FPKM for linked genes. Compare anchor signals and linked-promoter FPKM against Nott et al. epigenomic and Zhang et al. expression reference datasets, computing matched vs. unmatched cell type enrichment; summarize as fig2b_cell_type_loop_enrichment.tsv.

## Related tools

- **SnapHiC** (Identifies cell type-specific chromatin loops (loop summits in *.postprocessed.summits.bedpe) from scHi-C contact files per cell type; its fdr_dist column supplies the loop strength measure used for enrichment testing) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Quantifies CPM-normalized ATAC-seq/H3K27ac at loop anchors and FPKM for linked promoters, and computes matched vs. unmatched enrichment statistics)

## Examples

```
python snap.py --step postprocess --indir contacts_Astro/ --outdir out_Astro/ --genome hg19 --binsize 10000 --filter_file ext/hg19_filter_regions.txt --prefix Astro
```

## Evaluation signals

- Loop anchors show significantly higher ATAC-seq and H3K27ac signal in the matched cell type than in the other three cell types.
- Genes whose promoters are linked to cell type-specific loops show significantly higher FPKM expression in the matched cell type, reproducing Fig. 2b.
- Each summit bedpe contains non-empty fdr_dist values usable as a local-background-normalized loop strength ranking.
- All 10KB bins overlapping ENCODE blacklist regions or with mappability <=0.8 are absent from the loop calls.
- Output fig2b_cell_type_loop_enrichment.tsv exists with one enrichment row per cell type and consistently positive matched-vs-unmatched signal ratios.

## Limitations

- SnapHiC loop calling and the enrichment test depend on equal cell counts per type (261 cells here); unbalanced subsets confound the matched-vs-unmatched contrast.
- Loop calls are limited to 10KB-binned autosomes with hg19/mm10/hg38 support only, and use MboI-based mappability; other resolutions or restriction enzymes require separately downloaded genomic feature files.
- Default 'inverse' RWR solution is memory-intensive; the sliding_window alternative trades accuracy for memory.
- Reference epigenomic/expression datasets (Nott et al., Zhang et al.) must match the species and cell type taxonomy of the scHi-C subsets.

## Evidence

- [results] Anchors of cell type-specific loops showed significantly higher ATAC-seq and H3K27ac ChIP-seq signals in the matched cell type compared to other cell types: [evidence span withheld — anchor status: not_found]
- [results] using the fdr_dist column (false discovery rate with respect to local background at the same 1D genomic distance) as the measure of loop strength: [evidence span withheld — anchor status: not_found]
- [methods] We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions.: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [methods] SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR): "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR)"
- [readme] The following three input files are mandatory for SnapHiC: "The following three input files are mandatory for SnapHiC"
