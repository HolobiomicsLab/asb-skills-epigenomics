---
name: hmm-differential-call-post-processing
description: Use when you have histoneHMM differential calls for a broad histone mark
  (e.g. H3K9me3 female-vs-mouse male) and need to evaluate them against a defined
  gene set by base-pair and gene-count overlap.
when_to_use_negative:
- The input is single-sample modified-region calls rather than two-sample differential
  calls from get.differential.regions
- The target annotation includes exons/UTRs and the benchmark requires gene-body-only
  overlaps, so raw exon-level annotation would change the base-pair denominators
- The goal is peak-like (narrow) mark analysis; the article notes methods searching
  for peak-like features on broad marks such as H3K27me3/H3K9me3 generate false positives/negatives
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: histoneHMM
  role: Produces the differential region calls (get.differential.regions) that are
    intersected with gene bodies; provides bed2GR and gff2GR helpers
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which histoneHMM runs and where GenomicRanges overlap
    counting is performed
- name: Bioconductor
  role: Source of the GenomicRanges infrastructure used for base-pair and gene overlap
    counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:eaee496d87f78f71ccc2ba97ac69a46f704bfb50548e88e1eb2d45931c35389a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# hmm-differential-call-post-processing

## Summary

Post-process histoneHMM differential region calls (from get.differential.regions, exported as BED) by converting them to GenomicRanges with bed2GR and intersecting them with gene annotations loaded via gff2GR to quantify functional overlap, e.g. percent of X-inactivated gene base pairs and genes called differentially modified.

## When to use

Use when you have histoneHMM differential calls for a broad histone mark (e.g. H3K9me3 female-vs-mouse male) and need to evaluate them against a defined gene set by base-pair and gene-count overlap.

## When NOT to use

- The input is single-sample modified-region calls rather than two-sample differential calls from get.differential.regions
- The target annotation includes exons/UTRs and the benchmark requires gene-body-only overlaps, so raw exon-level annotation would change the base-pair denominators
- The goal is peak-like (narrow) mark analysis; the article notes methods searching for peak-like features on broad marks such as H3K27me3/H3K9me3 generate false positives/negatives

## Inputs

- histoneHMM differential region calls (BED output of get.differential.regions) for mouse female-vs-male H3K9me3
- Differential calls from competing methods (Diffreps, Chipdiff, Pepr, Rseg) in comparable region format
- Gene annotation in GFF format with gene bodies only, no exon information (e.g. ensembl59-genes.gff)
- List of 250 known X-inactivated protein coding genes

## Outputs

- Percent of X-inactivated gene base pairs called differentially modified per method (e.g. 67% for histoneHMM)
- Percent of the 250 X-inactivated genes with at least one differential bin per method (e.g. 99% for histoneHMM)
- Comparison table (table4_x_inactivation_overlap.tsv) with one row per method

## How to apply

Run histoneHMM's get.differential.regions on the binned read-count input to obtain differentially modified regions, then convert the resulting BED calls to GenomicRanges with bed2GR. Load a gene annotation in GFF format (e.g. the ensembl59-genes.gff provided by the project, gene bodies only, no exon information) with gff2GR and subset it to the target gene set such as the 250 known X-inactivated protein coding genes. Use GenomicRanges overlaps to count base pairs covered and the number of gene bodies with at least one differential bin, computing fractions of total target base pairs and of the 250 genes. Repeat identically on Diffreps, Chipdiff, Pepr and Rseg calls so comparison columns share the same intersection logic, then assemble a summary table (e.g. table4_x_inactivation_overlap.tsv).

## Related tools

- **histoneHMM** (Produces the differential region calls (get.differential.regions) that are intersected with gene bodies; provides bed2GR and gff2GR helpers) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which histoneHMM runs and where GenomicRanges overlap counting is performed)
- **Bioconductor** (Source of the GenomicRanges infrastructure used for base-pair and gene overlap counting)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); diffr <- get.differential.regions(...); gr <- bed2GR(diffr); genes <- gff2GR("ensembl59-genes.gff"); hits <- subsetByOverlaps(genes, gr)
```

## Evaluation signals

- histoneHMM column of the reproduced table matches benchmark values of 67% differential base pairs and 99% differential genes among the 250 X-inactivated protein coding genes
- Gene annotation loaded via gff2GR is restricted to gene bodies only (no exon information), matching the ensembl59-genes.gff used by the authors
- Denominator is exactly the 250 known X-inactivated protein coding genes; gene counts must be integers in [0, 250] and base-pair fractions in [0, 1]
- All competing methods (Diffreps, Chipdiff, Pepr, Rseg) are intersected with the same gene-body ranges and the same overlap logic, so columns are directly comparable
- histoneHMM's inactivated-gene percentage (99%) should substantially exceed Diffreps, Chipdiff and Pepr as reported

## Limitations

- Benchmark values (67% base pairs, 99% genes) depend on the specific 250-gene X-inactivation list and the gene-body-only ensembl59-genes.gff annotation; other gene sets or annotations will not reproduce them
- Competing methods were run with mostly default parameters and thresholds as recommended by the authors, and no version numbers are given for Chipdiff, rseg, Diffreps, Pepr, Sicer or Broadpeak, so exact reproduction may vary
- histoneHMM binning defaults to 1000 bp bins; gene overlap counts are at bin resolution, not single-nucleotide resolution

## Evidence

- [discussion] histoneHMM correctly identifies 62% of inactivated basepairs as differentially modified corresponding to 99% of inactivated genes, which is substantially more than what is reported by Diffreps, Chipdiff and Pepr.: "histoneHMM correctly identifies 62% of inactivated basepairs as differentially modified corresponding to 99% of inactivated genes"
- [discussion] partition the genome into 1000 bp bins: "we partition the genome into m equally sized bins (1000 bp by default)"
- [readme] GFF gene annotation with gene bodies only: "[ensembl59-genes.gff](http://histonehmm.molgen.mpg.de/data/ensembl59-genes.gff) : gene annotation in gff format (only gene bodies, no exon information)"
- [discussion] methods run with default parameters: "We mostly used the default parameters and thresholds as recommended by the authors since these results are likely those that an end user would also obtain"
