---
name: differential-expression-gene-list-processing
description: Use when RNA-seq gene expression data must be converted into a differentially
  expressed gene set and tested for significant overlap with differential histone
  modification calls (e.g. H3K27me3 or H3K9me3).
when_to_use_negative:
- Input ChIP-seq data contains peak-like histone marks rather than broad diffuse modifications
  such as H3K27me3/H3K9me3, where peak-calling methods are adequate and histoneHMM's
  broad-footprint modeling is unnecessary
- The task is only to call modified/differentially modified regions from ChIP-seq
  read data without any matched gene expression data to overlap against
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: histoneHMM
  role: Provides get.differential.regions to obtain differentially modified regions
    that are intersected with the differentially expressed gene list
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Analysis environment; stats::fisher.test computes overlap significance and
    the gene expression table is processed in R
- name: Bioconductor
  role: Bioinformatic tool sets (e.g. DESeq-based differential expression and annotation
    handling) integrated with histoneHMM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:3cf41c39cfb06d92728b80536256ed1ff7158b2a839811d86d96a1923d6b9ec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# differential-expression-gene-list-processing

## Summary

Derive a differentially expressed gene list from RNA-seq expression data (DESeq FDR<0.01), map it and histoneHMM differential modification calls to a common gene identifier via GFF annotation, and test their overlap with Fisher's exact test plus GO enrichment to validate differential modification calls.

## When to use

Use when RNA-seq gene expression data must be converted into a differentially expressed gene set and tested for significant overlap with differential histone modification calls (e.g. H3K27me3 or H3K9me3).

## When NOT to use

- Input ChIP-seq data contains peak-like histone marks rather than broad diffuse modifications such as H3K27me3/H3K9me3, where peak-calling methods are adequate and histoneHMM's broad-footprint modeling is unnecessary
- The task is only to call modified/differentially modified regions from ChIP-seq read data without any matched gene expression data to overlap against

## Inputs

- RNA-seq gene expression table (expression.txt)
- histoneHMM differential modification calls for rat H3K27me3 (SHR vs BN) and mouse H3K9me3 comparisons
- Gene annotation in GFF format (ensembl59-genes.gff; gene bodies only, no exon information)

## Outputs

- Differentially expressed gene list at DESeq FDR<0.01
- 2x2 contingency tables of DE x differentially-modified overlap per comparison
- Fisher's exact test P-values for overlap significance per comparison
- GO enrichment P-value for overlap gene set (e.g. GO:0019882)
- fisher_overlap_pvalues.csv with one P-value row per test

## How to apply

Load the RNA-seq gene expression table (expression.txt) and derive the differentially expressed gene list at DESeq FDR<0.01. Load histoneHMM differential modification calls (rat SHR/BN H3K27me3 and mouse H3K9me3), regenerating region calls with get.differential.regions from the histoneHMM R package where needed. Map both gene sets to a common identifier using the gene annotation in GFF format (ensembl59-genes.gff, gene bodies only) and build 2x2 contingency tables of differential-expression x differential-modification overlap for each comparison. Run R stats::fisher.test on each table and optionally a GO enrichment test for a term of interest (e.g. GO:0019882 'antigen processing and presentation') on the overlap gene set; write all contingency tables and P-values to a CSV (fisher_overlap_pvalues.csv), one P-value row per test.

## Related tools

- **histoneHMM** (Provides get.differential.regions to obtain differentially modified regions that are intersected with the differentially expressed gene list) — https://github.com/matthiasheinig/histoneHMM
- **R** (Analysis environment; stats::fisher.test computes overlap significance and the gene expression table is processed in R)
- **Bioconductor** (Bioinformatic tool sets (e.g. DESeq-based differential expression and annotation handling) integrated with histoneHMM)

## Examples

```
devtools::install_github("matthiasheinig/histoneHMM"); ft <- fisher.test(table(de_gene, diff_mod_region))
```

## Evaluation signals

- Recomputed Fisher's exact P-value for rat H3K27me3 overlap matches the reported 3.36e-6 (order-of-magnitude agreement)
- Recomputed Fisher's exact P-value for mouse H3K9me3 overlap matches the reported 1.39e-6 (order-of-magnitude agreement)
- GO:0019882 enrichment P-value on the rat overlap gene set matches the reported 4.79e-7
- fisher_overlap_pvalues.csv exists and contains one P-value row per test, with the 2x2 contingency tables included
- DE gene list is reproducibly defined by the DESeq FDR<0.01 threshold on expression.txt

## Limitations

- Differential modification region calls must sometimes be regenerated with get.differential.regions, adding a dependency on the histoneHMM package version and HMM fits
- The provided gene annotation (ensembl59-genes.gff) contains only gene bodies without exon information, limiting promoter/TSS-level overlap analyses
- Validation is by order-of-magnitude agreement of P-values; exact reproduction may vary with input data versions and thresholds

## Evidence

- [results] histoneHMM yielded the most significant overlap between differentially modified and differentially expressed genes (P=3.36×10−6, Fisher's exact test, Figure 3b): [evidence span withheld — anchor status: not_found]
- [results] gene ontology analysis revealed enrichment for the GO term "antigen processing and presentation" (GO:0019882, P=4.79·10−7): [evidence span withheld — anchor status: not_found]
- [readme] [ensembl59-genes.gff](http://histonehmm.molgen.mpg.de/data/ensembl59-genes.gff) : gene annotation in gff format (only gene bodies, no exon information): "gene annotation in gff format (only gene bodies, no exon information)"
- [readme] [expression.txt](http://histonehmm.molgen.mpg.de/data/expression.txt): RNA-seq gene expression: "[expression.txt](http://histonehmm.molgen.mpg.de/data/expression.txt): RNA-seq gene expression"
