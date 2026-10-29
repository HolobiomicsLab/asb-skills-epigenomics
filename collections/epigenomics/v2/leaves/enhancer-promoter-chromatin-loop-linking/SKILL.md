---
name: enhancer-promoter-chromatin-loop-linking
description: Use when assigning target genes to enhancer-resident noncoding GWAS SNPs
  (e.g. neuropsychiatric SNPs in prefrontal cortex) using cell type-specific chromatin
  loops such as SnapHiC summit BEDPE files from scHi-C/snm3C-seq data.
when_to_use_negative:
- Your goal is to call chromatin loops from raw scHi-C contact files rather than to
  link already-called loops to SNPs — use the SnapHiC pipeline (bin/rwr/hic/interaction/postprocess
  steps) instead.
- SNPs are coding or fall outside active enhancer intervals of the cell types with
  available loop calls, so no enhancer-anchor overlap can be defined.
- Loop data and SNP/enhancer annotations come from mismatched genome builds (e.g.
  hg19 BEDPE vs hg38 SNP coordinates) without liftover.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SnapHiC
  role: Produces the cell type-specific chromatin loop summit files (*.postprocessed.summits.bedpe)
    whose anchors are intersected with SNP-containing enhancers and gene promoters
  repo: https://github.com/HuMingLab/SnapHiC
- name: pybedtools/bioframe
  role: Performs the BED interval overlaps between GWAS SNPs, enhancer intervals,
    loop anchors, and promoters
- name: Python >= 3.6
  role: Runtime for the interval-overlap and linkage-aggregation script
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:1807b4f711aa19d636770f43dbecf8b71afbd68015548b926b1c8d31deda81e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  - pybedtools/bioframe
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# enhancer-promoter-chromatin-loop-linking

## Summary

Links enhancer-resident noncoding GWAS SNPs to putative target genes by overlapping SNP-containing enhancers with cell type-specific chromatin loops (SnapHiC *.postprocessed.summits.bedpe) called from single-nucleus methyl-3C-seq data, yielding a deduplicated SNP-gene linkage table with supporting cell types.

## When to use

Use when assigning target genes to enhancer-resident noncoding GWAS SNPs (e.g. neuropsychiatric SNPs in prefrontal cortex) using cell type-specific chromatin loops such as SnapHiC summit BEDPE files from scHi-C/snm3C-seq data.

## When NOT to use

- Your goal is to call chromatin loops from raw scHi-C contact files rather than to link already-called loops to SNPs — use the SnapHiC pipeline (bin/rwr/hic/interaction/postprocess steps) instead.
- SNPs are coding or fall outside active enhancer intervals of the cell types with available loop calls, so no enhancer-anchor overlap can be defined.
- Loop data and SNP/enhancer annotations come from mismatched genome builds (e.g. hg19 BEDPE vs hg38 SNP coordinates) without liftover.

## Inputs

- BED file of enhancer-residing noncoding GWAS SNP coordinates (e.g. 3,471 SNPs)
- Active enhancer BED intervals per brain cell type
- SnapHiC postprocessed loop summit files per cell type (*.postprocessed.summits.bedpe) with fdr_dist scores
- Gene promoter coordinates (e.g. TSS +/- interval) per reference genome (hg19/hg38)

## Outputs

- Deduplicated SNP-gene linkage table (e.g. 788 linkages, 445 SNPs, 189 genes)
- Per-linkage supporting cell type annotation
- Validated cell type-specific examples such as APOE enhancer links in astrocytes

## How to apply

Load the noncoding GWAS SNP coordinates (e.g. 3,471 enhancer-residing SNPs) and intersect them with the active enhancer BED intervals of each brain cell type (excitatory, inhibitory, oligodendrocyte, astrocyte) using pybedtools or bioframe interval overlap. Load each cell type's SnapHiC loop summit file (*.postprocessed.summits.bedpe), using fdr_dist (false discovery rate against local background at the same 1D genomic distance) to rank loop strength. For each enhancer-overlapping SNP, keep loops whose one anchor overlaps the SNP-containing enhancer and whose other anchor overlaps a gene promoter, restricting the link to the cell type in which the loop was called. Aggregate enhancer-promoter links across cell types, deduplicate to unique SNP-gene pairs, and record which cell type(s) support each linkage; a correct run on this task recovers 788 SNP-gene linkages connecting 445 SNPs to 189 genes, including astrocyte-specific APOE connections for rs112481437 and rs138137383.

## Related tools

- **SnapHiC** (Produces the cell type-specific chromatin loop summit files (*.postprocessed.summits.bedpe) whose anchors are intersected with SNP-containing enhancers and gene promoters) — https://github.com/HuMingLab/SnapHiC
- **pybedtools/bioframe** (Performs the BED interval overlaps between GWAS SNPs, enhancer intervals, loop anchors, and promoters)
- **Python >= 3.6** (Runtime for the interval-overlap and linkage-aggregation script)

## Examples

```
import pybedtools; snps_in_enh = pybedtools.BedTool('gwas_snps.bed').intersect('astrocyte_enhancers.bed', u=True)
```

## Evaluation signals

- Linkage table contains exactly 788 SNP-gene linkages connecting 445 unique SNPs to 189 unique genes when run on the prefrontal cortex task inputs.
- APOE appears with astrocyte-specific enhancer connections to rs112481437 and rs138137383 in the output.
- Every emitted linkage has a SNP located inside an enhancer interval that overlaps one anchor of a SnapHiC loop, and the loop's other anchor overlaps a gene promoter.
- No linkage is attributed to a cell type in which the loop was not called; deduplication leaves unique SNP-gene pairs with a list of supporting cell types.
- Loop summits used are postprocessed (*.postprocessed.summits.bedpe) rather than raw candidates, and fdr_dist is available for every used loop.

## Limitations

- Loops are only as good as the SnapHiC calls: 10KB bins with mappability <=0.8 and bins overlapping ENCODE blacklist regions are excluded from loop calling, so SNP-enhancer links in filtered regions are impossible.
- Linkages are restricted to the cell type in which each loop was called, so cell types without matched loop data contribute no links.
- Results are anchored to the analyzed dataset (single-nucleus methyl-3C-seq data from 2,869 human prefrontal cortical cells) and to reference genomes supported by SnapHiC (mm10, hg19, hg38); other tissues or builds require re-derivation.
- Links are enhancer-promoter associations inferred from 3D proximity, not experimentally validated regulatory relationships.

## Evidence

- [abstract] which uncovers cell type-specific chromatin loops and predicts putative target genes for noncoding sequence variants associated with neuropsychiatric disorders: "which uncovers cell type-specific chromatin loops and predicts putative target genes for noncoding sequence variants associated with neuropsychiatric disorders"
- [other] Applying SnapHiC loops from matching cell types to 3,471 enhancer-residing GWAS SNPs defined 788 SNP-gene linkages: [evidence span withheld — anchor status: not_found]
- [readme] We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions.: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] It is the prefix of the output files, including *.postprocessed.summits.bedpe: "It is the prefix of the output files, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe, and *allChr.[hic/cool]*"
- [abstract] identify chromatin loops at high resolution and accuracy from scHi-C data: "we describe Single-Nucleus Analysis Pipeline for Hi-C (SnapHiC), a method that can identify chromatin loops at high resolution and accuracy from scHi-C data"
