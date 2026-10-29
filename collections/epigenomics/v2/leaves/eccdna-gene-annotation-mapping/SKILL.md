---
name: eccdna-gene-annotation-mapping
description: Use when a catalog of eccDNA genomic coordinates (BED-style intervals)
  must be mapped to overlapping genes from a reference annotation (e.g., GENCODE GRCh37)
  to build an eccDNA gene table with Re/Rg overlap fractions.
when_to_use_negative:
- Input is an eccDNA identifier/gene list without chromosomal coordinates — interval
  intersection requires genomic start/end positions.
- The goal is expression or clinical comparison of already-annotated eccDNA genes
  (e.g., TCGA vs. CCLE t-tests) — use the downstream expression-analysis skill instead
  of re-running annotation.
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: BedTools
  role: Interval intersection of the eccDNA catalog against gene annotation using
    the -wo (write overlap) option to obtain per-pair overlap sizes
- name: AmpliconArchitect
  role: Upstream tool that identified the eccDNA elements from WGS data, producing
    the eccDNA coordinate catalog being annotated
- name: eccDNAdb scripts
  role: Source code repository containing the numbered analysis scripts for eccDNA
    identification and eccDNA-derived gene annotation
  repo: https://github.com/solo7773/eccDNAdb
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:4bea3498454b7a71688ff13f7142267b42363105a15a0b73dbde4890ae160f93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solo7773/eccDNAdb
  tools:
  - AmpliconArchitect
  - BedTools
  - eccDNAdb scripts
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# eccdna-gene-annotation-mapping

## Summary

Annotate identified eccDNA elements with genes by interval intersection of the eccDNA catalog against comprehensive gene annotation, computing per-pair overlap ratios Re (fraction of the eccDNA covered by the gene) and Rg (fraction of the gene covered by the eccDNA), yielding the eccDNA-derived gene set for downstream functional annotation as in eccDNAdb.

## When to use

Use when a catalog of eccDNA genomic coordinates (BED-style intervals) must be mapped to overlapping genes from a reference annotation (e.g., GENCODE GRCh37) to build an eccDNA gene table with Re/Rg overlap fractions.

## When NOT to use

- Input is an eccDNA identifier/gene list without chromosomal coordinates — interval intersection requires genomic start/end positions.
- The goal is expression or clinical comparison of already-annotated eccDNA genes (e.g., TCGA vs. CCLE t-tests) — use the downstream expression-analysis skill instead of re-running annotation.

## Inputs

- eccDNA catalog with chromosomal coordinates of identified eccDNA elements (e.g., from AmpliconArchitect analysis of WGS data or literature curation)
- GENCODE comprehensive gene annotation, GRCh37 release 38, converted to BED interval format

## Outputs

- eccdna_gene_annotation.tsv: tab-delimited table with columns gene, eccDNA, overlap size, Re, Rg (Re and Rg in [0, 1])
- Annotated eccDNA gene set (e.g., 54,901 genes in eccDNAdb) for downstream expression, clinical, and chromatin-accessibility analyses

## How to apply

Convert the eccDNA catalog and the GENCODE GRCh37 (release 38) comprehensive gene annotation into compatible BED interval files. Run BedTools intersect with the -wo option so that every eccDNA–gene overlap pair reports the size of the shared interval. For each pair compute Re = overlap size / eccDNA interval size and Rg = overlap size / gene interval size, both bounded in [0, 1]. Aggregate per gene and per eccDNA to produce the gene-annotation table used in eccDNAdb, where 54,901 eccDNA genes were annotated with an average of 7 genes per eccDNA. This annotated gene set then feeds downstream analyses such as TCGA/CCLE expression and t-tests, so correct interval arithmetic matters for everything built on top of it.

## Related tools

- **BedTools** (Interval intersection of the eccDNA catalog against gene annotation using the -wo (write overlap) option to obtain per-pair overlap sizes)
- **AmpliconArchitect** (Upstream tool that identified the eccDNA elements from WGS data, producing the eccDNA coordinate catalog being annotated)
- **eccDNAdb scripts** (Source code repository containing the numbered analysis scripts for eccDNA identification and eccDNA-derived gene annotation) — https://github.com/solo7773/eccDNAdb

## Examples

```
bedtools intersect -a eccdna_catalog.bed -b gencode_v38_genes.bed -wo | awk -F'\t' '{re=$NF/($2... ); print}' > eccdna_gene_annotation.tsv
```

## Evaluation signals

- eccdna_gene_annotation.tsv exists and is well-formed tab-delimited text with the required gene, eccDNA, overlap, Re, and Rg columns.
- All Re and Rg values fall within the range [0, 1] for every eccDNA–gene pair.
- One row exists per overlapping eccDNA–gene pair, with overlap size never exceeding either the eccDNA or the gene interval length.
- Per-eccDNA gene counts are populated and aggregate to the expected gene set (54,901 annotated eccDNA genes in the published build, averaging 7 genes per eccDNA).
- Downstream expression tables for annotated genes (e.g., hsa_Chr7_3S_1 genes in TCGA-GBM) are populated, confirming annotation feeds correctly into expression analyses.

## Limitations

- All eccDNAs in the published database were identified by the AmpliconArchitect pipeline only, so the annotated gene set inherits that pipeline's sensitivity and specificity limits.
- Gene counts vary among eccDNAs (average of 7 genes per eccDNA); large eccDNAs can capture many genes, inflating downstream gene-set sizes.
- Annotation is based on GENCODE GRCh37 (release 38) coordinates, so eccDNA intervals and annotation must share a consistent reference build before intersecting.

## Evidence

- [abstract] The gene-annotation step of eccDNAdb produced 54,901 annotated eccDNA genes, with gene counts varying among eccDNAs and an average of 7 genes per eccDNA.: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [readme] The README describes the repository purpose and the numbered-script convention for reproducing the annotation analyses.: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [discussion] AmpliconArchitect is the upstream eccDNA detection tool whose output catalog is annotated with genes.: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [discussion] EccDNA genes identified by annotation were subsequently used with TCGA and CCLE expression data.: "EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases."
