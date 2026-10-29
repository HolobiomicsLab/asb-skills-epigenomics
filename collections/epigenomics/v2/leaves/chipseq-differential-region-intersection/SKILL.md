---
name: chipseq-differential-region-intersection
description: Use when you have differential region calls from two or more ChIP-seq
  peak/broad-mark callers on the same genome assembly and an independent ground-truth
  coordinate set (qPCR-validated regions, X-inactivated gene bodies) to score sensitivity
  and false discovery per method.
when_to_use_negative:
- Input is raw ChIP-seq reads or BAM files with no differential calls yet produced
  — run a caller such as histoneHMM first
- The ground truth is a continuous track or signal profile rather than a discrete
  coordinate set suitable for GenomicRanges overlap operations
- Region calls and reference coordinates are on different genome assemblies — intersecting
  without lifting over to a common assembly (e.g. rn4) will produce spurious labels
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Source of differential region calls via get.differential.regions; benchmarked
    method
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment for the intersection workflow
- name: Bioconductor
  role: Provides GenomicRanges for overlap operations and bed2GR/gff2GR converters
- name: Diffreps
  role: Competing caller whose differential calls are intersected against the ground-truth
    regions
- name: Chipdiff
  role: Competing caller whose differential calls are intersected against the ground-truth
    regions
- name: Rseg
  role: Competing caller whose differential calls are intersected against the ground-truth
    regions
- name: Pepr
  role: Competing caller whose differential calls are intersected against the gene-body
    benchmark
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:b144f55e4861f7470633325613be7d222f5e94a7707c76077b254abdc1a4e64e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - Chipdiff
  - Diffreps
  - Pepr
  - R
  - Rseg
  - histoneHMM
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# chipseq-differential-region-intersection

## Summary

Benchmark ChIP-seq differential modification calls by intersecting each method's called regions (histoneHMM, Diffreps, Chipdiff, Rseg, Pepr) with independent coordinate sets — qPCR-surveyed regions or annotated gene bodies — using GenomicRanges overlap operations, then labeling each overlap as Y, FP, or FN.

## When to use

Use when you have differential region calls from two or more ChIP-seq peak/broad-mark callers on the same genome assembly and an independent ground-truth coordinate set (qPCR-validated regions, X-inactivated gene bodies) to score sensitivity and false discovery per method.

## When NOT to use

- Input is raw ChIP-seq reads or BAM files with no differential calls yet produced — run a caller such as histoneHMM first
- The ground truth is a continuous track or signal profile rather than a discrete coordinate set suitable for GenomicRanges overlap operations
- Region calls and reference coordinates are on different genome assemblies — intersecting without lifting over to a common assembly (e.g. rn4) will produce spurious labels

## Inputs

- Differential region calls from histoneHMM (output of get.differential.regions, default posterior-probability cutoff)
- Differential region calls from Diffreps, Chipdiff, Rseg, and Pepr (default author-recommended parameters)
- qPCR-surveyed region coordinates (e.g. the 11 H3K27me3 SHR vs BN regions from Table 3) on the rn4 rat genome
- Gene annotation in GFF format (gene bodies only, no exon information), e.g. ensembl59-genes.gff
- Known ground-truth gene sets, e.g. the 250 known X-inactivated protein coding genes

## Outputs

- An 11 rows x 4 methods Y/FP/FN label matrix written to qpcr_region_detection_table.csv
- Table of per-method percentages of differential base pairs and genes (e.g. table4_x_inactivation_overlap.tsv)
- Per-method overlap counts (overlapping base pairs, number of genes with at least one differential bin)

## How to apply

Load each method's called regions — histoneHMM differential regions from get.differential.regions with its default cutoff on the posterior probability, and Diffreps, Chipdiff, Rseg or Pepr calls run with default parameters recommended by their authors — into GRanges on a common assembly (e.g. rat rn4 or mouse). Convert annotations with bed2GR (BED) or gff2GR (GFF gene bodies, no exon information). Intersect the ground-truth coordinates with each method's call set using GenomicRanges overlap operations, labeling each method-region pair as Y (call overlaps the supported region), FP (call unsupported), or FN (supported region missed). For gene-body benchmarks, count overlapping base pairs and the number of genes with at least one differential bin, then express results as fractions of total ground-truth base pairs and genes. Assemble the per-method label matrix or percentage table and compare cell-by-cell against the published reference.

## Related tools

- **histoneHMM** (Source of differential region calls via get.differential.regions; benchmarked method) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment for the intersection workflow)
- **Bioconductor** (Provides GenomicRanges for overlap operations and bed2GR/gff2GR converters)
- **Diffreps** (Competing caller whose differential calls are intersected against the ground-truth regions)
- **Chipdiff** (Competing caller whose differential calls are intersected against the ground-truth regions)
- **Rseg** (Competing caller whose differential calls are intersected against the ground-truth regions)
- **Pepr** (Competing caller whose differential calls are intersected against the gene-body benchmark)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); d <- get.differential.regions(...); hits <- findOverlaps(bed2GR(d), gff2GR("ensembl59-genes.gff"))
```

## Evaluation signals

- Reproduced 11 x 4 Y/FP/FN label matrix matches published Table 3 cell-by-cell (output_matches_reference on all 44 labels)
- histoneHMM gene-body benchmark column matches reported 62% of X-inactivated base pairs / 99% of the 250 genes differential (67% base pairs cited in the validation criterion)
- Every ground-truth region receives exactly one label per method (Y, FP, or FN) with no missing or double-assigned entries
- All comparisons run on a single consistent genome assembly (rn4 for the rat qPCR benchmark)
- Chipdiff and Rseg recover only 5 and 6 of the validated differential regions respectively, consistent with their higher false negative rates reported in the article

## Limitations

- Competing methods (Chipdiff, Rseg, Diffreps, Pepr) were run with default parameters and thresholds as recommended by the authors, so results reflect default end-user settings rather than tuned optima
- Version numbers for Chipdiff, Rseg, Diffreps, Pepr, Sicer, Broadpeak, and samtools are not stated, affecting exact reproducibility
- Peak-like-feature methods generate many false positive or false negative calls on broad diffuse histone modifications such as H3K27me3, so labels are sensitive to caller choice
- Gene-body annotations used are gene bodies only, with no exon information

## Evidence

- [other] Label each method-region pair as Y, FP or FN: [evidence span withheld — anchor status: not_found]
- [discussion] Chipdiff and Rseg false negatives: [evidence span withheld — anchor status: not_found]
- [discussion] histoneHMM gene-body overlap performance: "histoneHMM correctly identifies 62% of inactivated basepairs as differentially modified corresponding to 99% of inactivated genes, which is substantially more than what is reported by Diffreps,"
- [discussion] Default parameters rationale: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [intro] Peak-method failure mode on broad marks: "Application of methods that search for peak-like features in such data can generate many false positive or false negative calls."
- [readme] README gene annotation file: "[ensembl59-genes.gff](http://histonehmm.molgen.mpg.de/data/ensembl59-genes.gff) : gene annotation in gff format (only gene bodies, no exon information)"
