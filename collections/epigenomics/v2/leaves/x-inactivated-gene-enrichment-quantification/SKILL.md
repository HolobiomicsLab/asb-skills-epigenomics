---
name: x-inactivated-gene-enrichment-quantification
description: Use when you have differential ChIP-seq region calls for a repressive
  broad mark (H3K9me3/H3K27me3) and a curated gene list (e.g. the 250 X-inactivated
  protein coding genes) and must report the percentage of overlapping base pairs and
  genes per method.
when_to_use_negative:
- Input is ChIP-seq of peak-like features (e.g. transcription factors) rather than
  broad diffuse marks such as H3K27me3 or H3K9me3, where methods searching for peaks
  were designed
- The goal is calling modified regions in a single sample rather than benchmarking
  two-sample differential calls against a gene set
- 'The comparison involves more than two samples: histoneHMM is a bivariate HMM comparing
  one experimental sample against one reference'
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: histoneHMM
  role: Bivariate HMM that produces the differential region calls being benchmarked
    against X-inactivated gene bodies
  repo: github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM and the GenomicRanges overlap analysis
    run
- name: Bioconductor
  role: Provides GenomicRanges infrastructure used to intersect differential bins
    with gene bodies from gff2GR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:b0ea63045d178b6889ad400eb1b572e3bd6b68fecf2dc9b783ef22e989f2552d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# x-inactivated-gene-enrichment-quantification

## Summary

Quantifies what fraction of known X-inactivated genes and their base pairs are called differentially modified (e.g. H3K9me3 between female and male mice) by histoneHMM versus competitors (Diffreps, Chipdiff, Pepr, Rseg), reproducing the paper's functional-benchmark table (Table 4).

## When to use

Use when you have differential ChIP-seq region calls for a repressive broad mark (H3K9me3/H3K27me3) and a curated gene list (e.g. the 250 X-inactivated protein coding genes) and must report the percentage of overlapping base pairs and genes per method.

## When NOT to use

- Input is ChIP-seq of peak-like features (e.g. transcription factors) rather than broad diffuse marks such as H3K27me3 or H3K9me3, where methods searching for peaks were designed
- The goal is calling modified regions in a single sample rather than benchmarking two-sample differential calls against a gene set
- The comparison involves more than two samples: histoneHMM is a bivariate HMM comparing one experimental sample against one reference

## Inputs

- Differential region calls in BED-like format from histoneHMM get.differential.regions (female vs male mouse H3K9me3)
- Differential region calls from Diffreps, Chipdiff, Pepr and Rseg for the same comparison
- Mouse gene annotation in GFF format, gene bodies only without exon information (e.g. ensembl59-genes.gff)
- List of the 250 known X-inactivated protein coding genes

## Outputs

- Table 4 style matrix of percentage of differentially modified base pairs and percentage of differentially modified genes per method
- table4_x_inactivation_overlap.tsv

## How to apply

Load the female-vs-male H3K9me3 differential calls from get.differential.regions and convert them with bed2GR; load mouse gene annotation in GFF format (gene bodies only, no exon information, e.g. ensembl59-genes.gff) with gff2GR and restrict to the 250 known X-inactivated protein coding genes. Intersect the histoneHMM differential regions (called on 1000 bp bins) with these gene bodies using GenomicRanges to count overlapping base pairs and the number of genes with at least one differential bin, then divide by the total inactivated base pairs and by 250. Repeat the same overlap for the Diffreps, Chipdiff, Pepr and Rseg calls to fill the comparison columns. The rationale is that true X-inactivated gene bodies should be differentially marked between sexes, so a method recovering a larger fraction of them is more functionally accurate; histoneHMM reports 62% of inactivated base pairs (99% of inactivated genes), substantially more than Diffreps, Chipdiff and Pepr.

## Related tools

- **histoneHMM** (Bivariate HMM that produces the differential region calls being benchmarked against X-inactivated gene bodies) — github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM and the GenomicRanges overlap analysis run)
- **Bioconductor** (Provides GenomicRanges infrastructure used to intersect differential bins with gene bodies from gff2GR)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Validation: the histoneHMM column of the reproduced table must match the reported benchmark values (62% of inactivated base pairs / 67% differential base pairs per the card, and 99% differential genes) among the 250 X-inactivated protein coding genes
- Fractions are expressed out of exactly 250 genes; any value above 100% or a gene denominator other than 250 indicates a mis-specified annotation subset
- Competitor columns (Diffreps, Chipdiff, Pepr, Rseg) should come out substantially lower than histoneHMM, per the reported finding
- Output file table4_x_inactivation_overlap.tsv must contain both percentage-of-base-pairs and percentage-of-genes columns for every method

## Limitations

- histoneHMM is designed for broad genomic footprints like H3K27me3; applying it to sharp, peak-like marks is out of scope
- No version numbers were stated for Chipdiff, rseg, Diffreps, Pepr, Sicer and Broadpeak in the article, so exact reproduction of competitor columns is ambiguous
- The gene annotation used contains only gene bodies with no exon information, so overlap counts cannot resolve gene sub-features

## Evidence

- [results] histoneHMM correctly identifies 62% of inactivated basepairs as differentially modified corresponding to 99% of inactivated genes: "histoneHMM correctly identifies 62% of inactivated basepairs as differentially modified corresponding to 99% of inactivated genes, which is substantially more than what is reported by Diffreps"
- [discussion] we partition the genome into m equally sized bins (1000 bp by default): "we partition the genome into m equally sized bins (1000 bp by default)"
- [methods] devtools::install_github("matthiasheinig/histoneHMM"): "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3.: "histoneHMM is a software to analyse ChIP-seq data of histone modifications with broad genomic footprints like H3K27me3."
