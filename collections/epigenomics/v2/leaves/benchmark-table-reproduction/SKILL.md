---
name: benchmark-table-reproduction
description: 'Use when reproducing the paper''s functional-validation benchmarks:
  measuring what fraction of base pairs and genes of a known functional gene set (e.g.
  X-inactivated genes) are covered by differential modification calls.'
when_to_use_negative:
- The calls are peak-like transcription-factor ChIP-seq peaks rather than broad diffuse
  repressive marks such as H3K9me3 or H3K27me3
- You need exon-level or promoter-level overlap rather than whole gene-body coverage,
  since the provided GFF annotation contains only gene bodies and no exon information
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0091
tools:
- name: histoneHMM
  role: Produces the differential H3K9me3 region calls (get.differential.regions,
    bed2GR, gff2GR) evaluated in the benchmark table
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM runs and in which the GenomicRanges
    intersections are performed
- name: Bioconductor
  role: Tool set (GenomicRanges) used to intersect differential regions with gene
    bodies and count overlapping base pairs and genes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:3d0a103eadc6e57f5351fb1bdf94a13ffaf0fddd7ea105ca876e6a4ba08a5f6f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# benchmark-table-reproduction

## Summary

Reproduce the benchmark tables of the histoneHMM paper, notably Table 4's X-inactivation coverage evaluation of H3K9me3 differential calls, by intersecting differential region calls with the 250 known X-inactivated protein coding genes and comparing histoneHMM against Diffreps, Chipdiff, Pepr and Rseg.

## When to use

Use when reproducing the paper's functional-validation benchmarks: measuring what fraction of base pairs and genes of a known functional gene set (e.g. X-inactivated genes) are covered by differential modification calls.

## When NOT to use

- The calls are peak-like transcription-factor ChIP-seq peaks rather than broad diffuse repressive marks such as H3K9me3 or H3K27me3
- You need exon-level or promoter-level overlap rather than whole gene-body coverage, since the provided GFF annotation contains only gene bodies and no exon information

## Inputs

- Differential region calls in BED format from get.differential.regions (histoneHMM), plus Diffreps, Chipdiff, Pepr and Rseg calls for mouse female-vs-male H3K9me3
- Mouse gene annotation in GFF format (gene bodies only, no exon information), e.g. ensembl59-genes.gff
- List of the 250 known X-inactivated protein coding genes

## Outputs

- table4_x_inactivation_overlap.tsv with percentage of differential base pairs and percentage of the 250 genes covered, per method
- Per-method overlap counts (overlapping base pairs and number of genes with at least one differential bin) computed via GenomicRanges

## How to apply

Load the mouse female-vs-male H3K9me3 differential regions produced by get.differential.regions and convert them to genomic ranges with bed2GR. Load the gene annotation in GFF format with gff2GR, noting the annotation contains only gene bodies and no exon information, and restrict it to the 250 known X-inactivated protein coding genes. Intersect histoneHMM differential regions with the gene bodies using GenomicRanges to count overlapping base pairs and genes with at least one differential bin, then compute the fraction of differential base pairs and the fraction of the 250 genes. Repeat the same intersection for the Diffreps, Chipdiff, Pepr and Rseg calls, using default parameters and thresholds as recommended by the competing methods' authors, and assemble the percentages into the Table 4 layout.

## Related tools

- **histoneHMM** (Produces the differential H3K9me3 region calls (get.differential.regions, bed2GR, gff2GR) evaluated in the benchmark table) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM runs and in which the GenomicRanges intersections are performed)
- **Bioconductor** (Tool set (GenomicRanges) used to intersect differential regions with gene bodies and count overlapping base pairs and genes)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- The histoneHMM column of the reproduced table matches the reported 67% differential base pairs and 99% differential genes among the 250 X-inactivated protein coding genes
- The paper's finding holds: histoneHMM identifies substantially more inactivated base pairs/genes than Diffreps, Chipdiff and Pepr (62% of inactivated basepairs, 99% of inactivated genes per the finding card)
- Output table contains all comparison columns (histoneHMM, Diffreps, Chipdiff, Pepr, Rseg) with percentage values in valid 0-100 ranges
- Gene-body overlaps are computed against exactly 250 known X-inactivated protein coding genes, gene bodies only

## Limitations

- Competing method versions are unspecified: no version numbers are given for Chipdiff, rseg, Diffreps, Pepr, Sicer, Broadpeak
- The GFF gene annotation contains only gene bodies with no exon information, limiting resolution of the overlap analysis
- Competing methods were run mostly with default parameters and thresholds as recommended by the authors, so results may differ under alternative settings

## Evidence

- [abstract] histoneHMM outperforms competing methods in detecting functionally relevant differentially modified regions: "Our results show that histoneHMM outperforms competing methods in detecting functionally relevant differentially modified regions"
- [discussion] Default parameters for competing methods: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
- [discussion] Genome binning of reads: "we partition the genome into m equally sized bins (1000 bp by default)"
- [readme] GFF gene annotation with gene bodies only: "ensembl59-genes.gff : gene annotation in gff format (only gene bodies, no exon information)"
