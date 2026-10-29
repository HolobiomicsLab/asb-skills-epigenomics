---
name: go-term-enrichment-statistics
description: Use when you have a list of differentially modified genomic regions from
  a ChIP-seq differential caller and a list of differentially expressed genes, and
  you need to test overlap significance and GO term enrichment of the shared genes.
when_to_use_negative:
- Input is peak-like ChIP-seq data (e.g. sharp transcription factor peaks) rather
  than broad diffuse marks like H3K27me3/H3K9me3 — the histoneHMM calls feeding this
  overlap test were designed for broad genomic footprints
- Gene sets cannot be mapped to a common identifier via the available GFF annotation,
  making the contingency table undefined
- You need pathway-level effect sizes or ranked-gene tests rather than a binary DE
  x differential-modification overlap test
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: R
  role: runtime environment for the Fisher's exact test (stats::fisher.test) and GO
    enrichment statistics
- name: histoneHMM
  role: generates the differentially modified region calls (get.differential.regions)
    whose overlap with DE genes is tested
  repo: github.com/matthiasheinig/histoneHMM
- name: Bioconductor
  role: provides the bioinformatic tool sets (gene annotation and GO enrichment) the
    package integrates with
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:e3d383217c3b4b4eb9ae9b27c52753bc1c2e85bdeaba46a67f998c97a1aef95d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# go-term-enrichment-statistics

## Summary

A statistical skill for testing whether differentially modified histone regions (called by histoneHMM) overlap with differentially expressed genes more than expected, using Fisher's exact test on 2x2 contingency tables, followed by GO term enrichment (e.g. 'antigen processing and presentation', GO:0019882).

## When to use

Use when you have a list of differentially modified genomic regions from a ChIP-seq differential caller and a list of differentially expressed genes, and you need to test overlap significance and GO term enrichment of the shared genes.

## When NOT to use

- Input is peak-like ChIP-seq data (e.g. sharp transcription factor peaks) rather than broad diffuse marks like H3K27me3/H3K9me3 — the histoneHMM calls feeding this overlap test were designed for broad genomic footprints
- Gene sets cannot be mapped to a common identifier via the available GFF annotation, making the contingency table undefined
- You need pathway-level effect sizes or ranked-gene tests rather than a binary DE x differential-modification overlap test

## Inputs

- expression.txt — RNA-seq gene expression table
- histoneHMM differential modification region calls (rat H3K27me3, mouse H3K9me3)
- ensembl59-genes.gff — gene annotation in gff format (gene bodies only, no exon information)
- GO term of interest (GO:0019882)

## Outputs

- fisher_overlap_pvalues.csv containing one P-value row per test (overlap Fisher tests plus GO enrichment test)
- 2x2 contingency tables of DE x differentially-modified overlap for rat and mouse
- GO enrichment P-value for GO:0019882 (expected 4.79e-7)

## How to apply

Load the RNA-seq data (expression.txt) and derive the differentially expressed gene list at DESeq FDR < 0.01. Load the histoneHMM differential modification calls (e.g. rat.H3K27me3, mouse H3K9me3), regenerating them with get.differential.regions from the histoneHMM R package if needed. Map both gene sets to a common identifier using the gene annotation in gff format (ensembl59-genes.gff) and build 2x2 contingency tables of DE x differentially-modified overlap. Run R stats::fisher.test on each table and check the P-values against the reported values 3.36e-6 (rat H3K27me3) and 1.39e-6 (mouse H3K9me3). For the rat overlap gene set, run a GO enrichment test for term GO:0019882 and verify the reported P of 4.79e-7; write all contingency tables and P-values to fisher_overlap_pvalues.csv.

## Related tools

- **R** (runtime environment for the Fisher's exact test (stats::fisher.test) and GO enrichment statistics)
- **histoneHMM** (generates the differentially modified region calls (get.differential.regions) whose overlap with DE genes is tested) — github.com/matthiasheinig/histoneHMM
- **Bioconductor** (provides the bioinformatic tool sets (gene annotation and GO enrichment) the package integrates with)

## Examples

```
p <- fisher.test(table(factor(genes$de.fdr0.01), factor(genes$diffmod.h3k27me3)))$p.value  # expect ~3.36e-6
```

## Evaluation signals

- Recomputed Fisher's exact P-values agree with the reported 3.36e-6 (rat H3K27me3) and 1.39e-6 (mouse H3K9me3) at order-of-magnitude agreement
- GO:0019882 enrichment P matches the reported 4.79e-7
- fisher_overlap_pvalues.csv exists and contains one P-value row per test (two overlap tests plus GO test)
- Each 2x2 contingency table's cells sum to the total number of mapped genes, confirming the identifier mapping covered both gene sets consistently
- DE gene list is reproducibly derived at DESeq FDR < 0.01 before building the tables

## Limitations

- Validation uses order-of-magnitude agreement for the overlap P-values rather than exact reproduction
- The ensembl59-genes.gff annotation contains only gene bodies with no exon information, which constrains identifier mapping precision
- histoneHMM version and archival DOI for the software are not stated; some supplementary details (e.g. Additional file 1) are not included in the article text

## Evidence

- [full_text] histoneHMM yielded the most significant overlap between differentially modified and differentially expressed genes (P=3.36×10−6, Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [full_text] gene ontology analysis revealed enrichment for the GO term antigen processing and presentation: [evidence span withheld — anchor status: not_found]
- [readme] expression.txt RNA-seq gene expression: "[expression.txt](http://histonehmm.molgen.mpg.de/data/expression.txt): RNA-seq gene expression"
- [readme] ensembl59-genes.gff gene annotation in gff format (only gene bodies, no exon information): "[ensembl59-genes.gff](http://histonehmm.molgen.mpg.de/data/ensembl59-genes.gff) : gene annotation in gff format (only gene bodies, no exon information)"
- [intro] histoneHMM seamlessly integrates with Bioconductor tool sets: "seamlessly integrates with the extensive bioinformatic tool sets available through Bioconductor"
- [methods] histoneHMM installation via devtools: "devtools::install_github("matthiasheinig/histoneHMM")"
