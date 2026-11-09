---
name: gwas-variant-target-gene-prediction
description: Use when you have enhancer-annotated noncoding GWAS SNP coordinates and
  cell type-specific chromatin loop files (*.postprocessed.summits.bedpe) and need
  SNP-gene linkages.
when_to_use_negative:
- The GWAS variants of interest are coding variants where direct protein-coding consequences,
  not enhancer-promoter looping, explain the association
- Only bulk Hi-C data are available; SnapHiC loop calling requires single-cell Hi-C
  contact files per cell
- The cell type from which loops were called does not match the disease-relevant cell
  type, breaking the cell type-restricted linkage logic
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0622
tools:
- name: SnapHiC
  role: Produces the cell type-specific chromatin loop summit files (*.postprocessed.summits.bedpe)
    that anchor SNP-gene linkages
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Runtime for interval-overlap scripting (pybedtools/bioframe) to intersect
    SNPs, enhancers, loop anchors, and promoters
  repo: https://github.com/HuMingLab/SnapHiC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:0278cb06be314e65864e96547f727ed8d7e596196fedd39eca93464dbbd2e7aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gwas-variant-target-gene-prediction

## Summary

Predicts putative target genes for noncoding GWAS variants by intersecting enhancer-resident SNPs with cell type-specific chromatin loops, connecting disease-associated SNPs to promoters of putative target genes.

## When to use

Use when you have enhancer-annotated noncoding GWAS SNP coordinates and cell type-specific chromatin loop files (*.postprocessed.summits.bedpe) and need SNP-gene linkages.

## When NOT to use

- The GWAS variants of interest are coding variants where direct protein-coding consequences, not enhancer-promoter looping, explain the association
- Only bulk Hi-C data are available; SnapHiC loop calling requires single-cell Hi-C contact files per cell
- The cell type from which loops were called does not match the disease-relevant cell type, breaking the cell type-restricted linkage logic

## Inputs

- Noncoding GWAS SNP coordinate table (e.g. 3,471 enhancer-residing SNP positions)
- Active enhancer BED intervals per brain cell type (four cell types)
- SnapHiC loop summit files (*.postprocessed.summits.bedpe) per cell type with fdr_dist loop strength
- Gene promoter coordinate BED file

## Outputs

- Deduplicated SNP-gene linkage table (788 linkages, 445 SNPs, 189 genes) with supporting cell type annotations
- Cell type-specific enhancer-promoter loop links connecting GWAS SNPs to putative target genes

## How to apply

Load the noncoding GWAS SNP coordinates (e.g. 3,471 enhancer-residing SNPs) and intersect them with active enhancer BED intervals of each brain cell type using pybedtools/bioframe interval overlap. Load the cell type-specific SnapHiC loop summit files (*.postprocessed.summits.bedpe), using fdr_dist (false discovery rate with respect to local background at the same 1D genomic distance) as the loop strength measure. For each enhancer-overlapping SNP, identify loops whose one anchor overlaps the SNP-containing enhancer and whose other anchor overlaps a gene promoter, restricting links to the cell type in which the loop was called. Aggregate enhancer-promoter loop links across the four cell types, deduplicate to unique SNP-gene pairs, and record the supporting cell type(s); expect 788 linkages connecting 445 SNPs to 189 genes, including APOE astrocyte-specific connections for rs112481437 and rs138137383.

## Related tools

- **SnapHiC** (Produces the cell type-specific chromatin loop summit files (*.postprocessed.summits.bedpe) that anchor SNP-gene linkages) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Runtime for interval-overlap scripting (pybedtools/bioframe) to intersect SNPs, enhancers, loop anchors, and promoters) — https://github.com/HuMingLab/SnapHiC

## Examples

```
import bioframe; enh_snp = bioframe.overlap(snps, enhancers); links = bioframe.overlap(enh_snp, loops, suffixes=('_snp','_loop'))
```

## Evaluation signals

- The linkage table contains exactly 788 SNP-gene linkages connecting 445 disease-associated SNPs to 189 genes
- APOE shows astrocyte-specific enhancer connections for rs112481437 and rs138137383
- Every reported linkage is supported by a loop whose one anchor overlaps the SNP-containing enhancer and whose other anchor overlaps a gene promoter
- Deduplication yields unique SNP-gene pairs with recorded supporting cell type(s) from the four brain cell types
- All retained loops carry fdr_dist values as the loop strength measure

## Limitations

- Loop linkage is restricted to the cell type in which the loop was called; cross-cell-type aggregation is only for deduplication, not transfer
- Linkages depend on enhancer annotations; SNPs outside active enhancer intervals in the four cell types are excluded
- SnapHiC loop candidates are postprocessed via clustering nearby candidates and identifying loop summits; summit resolution bounds anchor precision
- Bins with mappability <=0.8 or overlapping ENCODE blacklist regions are excluded from loop calling, limiting coverage in those regions

## Evidence

- [other] Applying SnapHiC loops from matching cell types to 3,471 enhancer-residing GWAS SNPs defined 788 SNP-gene linkages connecting 445 disease-associated SNPs to 189 genes: [evidence span withheld — anchor status: not_found]
- [abstract] cell type-specific loops predict target genes: "which uncovers cell type-specific chromatin loops and predicts putative target genes for noncoding sequence variants associated with neuropsychiatric disorders"
- [methods] SnapHiC five steps: "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR)"
- [methods] mappability filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] output file suffix: "It is the prefix of the output files, including *.postprocessed.summits.bedpe*, *.postprocessed.all_candidates.bedpe*"
