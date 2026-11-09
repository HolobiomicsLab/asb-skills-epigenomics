---
name: cell-type-specific-regulatory-variant-mapping
description: Use when mapping enhancer-resident noncoding GWAS SNPs to putative target
  genes via cell type-specific chromatin loops (SnapHiC *.postprocessed.summits.bedpe),
  e.g. neuropsychiatric SNPs in human prefrontal cortex.
when_to_use_negative:
- Calling chromatin loops de novo from single-cell Hi-C contact files — run the SnapHiC
  steps (bin, rwr, hic, interaction, postprocess) instead; this skill consumes already-called
  loop summits.
- Variants are coding or already carry eQTL/colocalization-based gene assignments,
  so enhancer-promoter loop linkage adds no new information.
- Only bulk (non-cell-type-resolved) chromatin contact data is available — the cell
  type-specific linkage restriction cannot be honored.
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3308
tools:
- name: SnapHiC
  role: Produces the cell type-specific chromatin loop summits (*.postprocessed.summits.bedpe)
    whose anchors are intersected with enhancer-resident GWAS SNPs and gene promoters
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Runtime for the pybedtools/bioframe interval-overlap scripts that intersect
    SNP coordinates, enhancer BEDs, and loop anchors
- name: pybedtools
  role: Interval overlap of the 3,471 GWAS SNP coordinates with the active enhancer
    BED intervals of each brain cell type
- name: bioframe
  role: Alternative interval-overlap backend for intersecting SNP-containing enhancers
    with loop anchors and promoters
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:e88bb5fc55f592e191a902edb752a6072ef45a796fd66628807bdfeb8ff0a12e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  - bioframe
  - pybedtools
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-type-specific-regulatory-variant-mapping

## Summary

Maps noncoding neuropsychiatric GWAS SNPs to putative target genes by intersecting enhancer-resident SNP coordinates with cell type-specific SnapHiC loop summits from single-nucleus methyl-3C-seq data of 2,869 human prefrontal cortical cells, yielding SNP-gene linkages each supported by a named brain cell type.

## When to use

Use when mapping enhancer-resident noncoding GWAS SNPs to putative target genes via cell type-specific chromatin loops (SnapHiC *.postprocessed.summits.bedpe), e.g. neuropsychiatric SNPs in human prefrontal cortex.

## When NOT to use

- Calling chromatin loops de novo from single-cell Hi-C contact files — run the SnapHiC steps (bin, rwr, hic, interaction, postprocess) instead; this skill consumes already-called loop summits.
- Variants are coding or already carry eQTL/colocalization-based gene assignments, so enhancer-promoter loop linkage adds no new information.
- Only bulk (non-cell-type-resolved) chromatin contact data is available — the cell type-specific linkage restriction cannot be honored.

## Inputs

- 3,471 noncoding neuropsychiatric GWAS SNP coordinates (BED-format interval table)
- Cell type-specific active enhancer BED intervals for each of the four brain cell types
- Cell type-specific SnapHiC loop summit files (*.postprocessed.summits.bedpe) with fdr_dist loop-strength scores
- Gene promoter coordinate intervals for overlapping the distal loop anchor
- chrom.sizes / reference genome build (hg19 or hg38) matching the loop calls

## Outputs

- Deduplicated SNP-gene linkage table (788 linkages connecting 445 SNPs to 189 genes)
- Per-linkage annotation of the supporting cell type(s) whose SnapHiC loop connects the SNP-containing enhancer to the promoter
- Cell type-specific APOE enhancer connection records for rs112481437 and rs138137383

## How to apply

Load the 3,471 noncoding GWAS SNP coordinates and intersect them with the active enhancer BED intervals of each of the four brain cell types using pybedtools/bioframe interval overlap, retaining only enhancer-resident SNPs. Load the cell type-specific SnapHiC loop summit files (*.postprocessed.summits.bedpe) — the output of the postprocess step that clusters nearby loop candidates and identifies loop summits — and use fdr_dist (false discovery rate with respect to local background at the same 1D genomic distance) as the loop-strength measure. For each enhancer-overlapping SNP, keep loops whose one anchor overlaps the SNP-containing enhancer and whose other anchor overlaps a gene promoter, restricting each link to the cell type in which the loop was called. Aggregate enhancer-promoter loop links across the four cell types, deduplicate to unique SNP-gene pairs, and record the cell type(s) supporting each linkage. Validate against the published result: 788 SNP-gene linkages connecting 445 SNPs to 189 genes, including astrocyte-specific APOE enhancer connections for rs112481437 and rs138137383.

## Related tools

- **SnapHiC** (Produces the cell type-specific chromatin loop summits (*.postprocessed.summits.bedpe) whose anchors are intersected with enhancer-resident GWAS SNPs and gene promoters) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Runtime for the pybedtools/bioframe interval-overlap scripts that intersect SNP coordinates, enhancer BEDs, and loop anchors)
- **pybedtools** (Interval overlap of the 3,471 GWAS SNP coordinates with the active enhancer BED intervals of each brain cell type)
- **bioframe** (Alternative interval-overlap backend for intersecting SNP-containing enhancers with loop anchors and promoters)

## Examples

```
import pybedtools; pybedtools.BedTool('gwas_snps.bed').intersect('astrocyte_enhancers.bed', wa=True, wb=True).saveas('snp_enhancer_overlaps.tsv')
```

## Evaluation signals

- Deduplicated linkage table contains exactly 788 SNP-gene linkages connecting 445 SNPs to 189 genes
- APOE shows astrocyte-specific enhancer connections for rs112481437 and rs138137383
- Every linkage records at least one supporting cell type, and the supporting loop was called in that same cell type
- Each linked SNP overlaps an active enhancer interval in the supporting cell type, and the distal loop anchor overlaps a gene promoter
- Each linkage traces back to a loop entry with an fdr_dist value in the matching *.postprocessed.summits.bedpe file

## Limitations

- Loop calling removes all 10KB bins with mappability <=0.8 and all bins overlapped with the ENCODE blacklist regions, so SNPs falling in filtered bins cannot be linked
- SnapHiC currently accepts only mm10, hg19 and hg38; SNP, enhancer, promoter, and loop coordinates must all be on one matching genome build
- Linkage resolution is bounded by the 10KB bin resolution of loop calling and by the postprocessed loop summits rather than raw contacts
- SnapHiC version 0.2.0 (October 29th, 2021) has no changelog in the repo, so version drift in loop summits could shift linkage counts

## Evidence

- [abstract] Cell type-specific loops predict target genes for noncoding neuropsychiatric variants: "which uncovers cell type-specific chromatin loops and predicts putative target genes for noncoding sequence variants associated with neuropsychiatric disorders"
- [abstract] Prefrontal cortex single-nucleus methyl-3C-seq data underlie the cell type-specific loops: "analyzing single-nucleus methyl-3C-seq data from 2,869 human prefrontal cortical cells, which uncovers cell type-specific chromatin loops"
- [methods] Loop calling excludes low-mappability and blacklist 10KB bins: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [methods] Postprocess step yields the loop summits used as linkage anchors: "(5) postprocessing: clustering nearby loop candidates and identifying loop summits"
- [readme] Loop summits are delivered as .postprocessed.summits.bedpe output files: "It is the prefix of the output files, including *.postprocessed.summits.bedpe*, *.postprocessed.all_candidates.bedpe*, and *allChr.[hic/cool]*."
- [readme] Genome build constraint for coordinates: "SnapHiC currently accepts mm10, hg19 and hg38. It is used to determine the number of autosomal chromosomes"
