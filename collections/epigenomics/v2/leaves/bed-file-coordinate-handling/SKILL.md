---
name: bed-file-coordinate-handling
description: Use when you have chromosomal coordinates of identified eccDNA elements
  and need to annotate overlapping genes, requiring BED interval conversion and overlap-fraction
  computation.
when_to_use_negative:
- You need to identify eccDNAs de novo from WGS data (e.g., running the AmpliconArchitect
  pipeline) rather than annotating genes on an existing eccDNA catalog
- Intervals cannot be converted to a common BED representation, e.g., one source is
  in 1-based GTF/GFF coordinates and no coordinate conversion is performed before
  intersect
- You only need visualization of eccDNA–gene relationships rather than quantitative
  overlap annotation
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BedTools
  role: 'Performs the interval intersection: bedtools intersect -wo yields per-pair
    overlap sizes between the eccDNA BED catalog and the gene annotation BED file'
- name: GENCODE
  role: Source of the comprehensive gene annotation (GRCh37, release 38) converted
    into a BED interval file for intersecting with the eccDNA catalog
- name: AmpliconArchitect
  role: Upstream tool that produced the eccDNA catalog (chromosomal coordinates of
    identified eccDNA elements) that this skill converts to BED for gene annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:cf1a2849036fb66518f03dfdf801c496799bc41cc81996e685ff731cc33335b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - BedTools
  - GENCODE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# bed-file-coordinate-handling

## Summary

Converts genomic interval catalogs (e.g., eccDNA coordinates) and gene annotations (e.g., GENCODE GRCh37 release 38) into compatible BED files and uses BedTools intersect -wo to quantify eccDNA–gene overlaps via the Re and Rg overlap ratios.

## When to use

Use when you have chromosomal coordinates of identified eccDNA elements and need to annotate overlapping genes, requiring BED interval conversion and overlap-fraction computation.

## When NOT to use

- You need to identify eccDNAs de novo from WGS data (e.g., running the AmpliconArchitect pipeline) rather than annotating genes on an existing eccDNA catalog
- Intervals cannot be converted to a common BED representation, e.g., one source is in 1-based GTF/GFF coordinates and no coordinate conversion is performed before intersect
- You only need visualization of eccDNA–gene relationships rather than quantitative overlap annotation

## Inputs

- eccDNA catalog as chromosomal coordinates of identified eccDNA elements
- GENCODE GRCh37 (release 38) comprehensive gene annotation
- BED-format interval files derived from both sources

## Outputs

- eccdna_gene_annotation.tsv: tab-delimited table with columns gene, eccDNA, overlap size, Re, Rg
- Annotated set of eccDNA genes (54,901 genes in the source study)

## How to apply

First convert both the eccDNA catalog (chromosomal coordinates of identified eccDNA elements) and the GENCODE GRCh37 (release 38) comprehensive gene annotation into compatible BED interval files, ensuring consistent coordinate conventions. Run BedTools intersect with the -wo (write overlap) option so that every eccDNA–gene pair is emitted together with the size of the overlapping genomic interval. For each overlap pair, compute Re as the overlap size divided by the eccDNA interval size and Rg as the overlap size divided by the gene interval size, then aggregate per gene into a tab-delimited table (gene, eccDNA, overlap size, Re, Rg) written to eccdna_gene_annotation.tsv. The rationale is that the -wo flag directly supplies the overlap lengths needed to express overlap as fractions of both intervals, matching how eccDNAdb annotated its 54,901 eccDNA genes.

## Related tools

- **BedTools** (Performs the interval intersection: bedtools intersect -wo yields per-pair overlap sizes between the eccDNA BED catalog and the gene annotation BED file)
- **GENCODE** (Source of the comprehensive gene annotation (GRCh37, release 38) converted into a BED interval file for intersecting with the eccDNA catalog)
- **AmpliconArchitect** (Upstream tool that produced the eccDNA catalog (chromosomal coordinates of identified eccDNA elements) that this skill converts to BED for gene annotation)

## Examples

```
bedtools intersect -wo -a eccdna_catalog.bed -b gencode_grch38_genes.bed > eccdna_gene_annotation.tsv
```

## Evaluation signals

- Output file eccdna_gene_annotation.tsv exists and is well-formed tab-delimited text with the required Re and Rg columns
- All Re and Rg values fall within the range [0, 1], since each is an overlap size divided by a containing interval size
- Number of distinct annotated genes is non-trivial and consistent with expectation (source study annotated 54,901 eccDNA genes, averaging ~7 genes per eccDNA)
- Every overlap size recorded is <= both the corresponding eccDNA interval size and the gene interval size
- eccDNA–gene pairs in the table match actual coordinate overlaps when spot-checked against the source BED files

## Limitations

- Ratios only capture partial-overlap semantics of bedtools intersect on the provided coordinates; results depend on the eccDNA catalog quality, which in eccDNAdb came solely from the AmpliconArchitect pipeline
- Gene annotation is genome-build dependent: the workflow specifically used GENCODE GRCh37 (release 38), so catalogs on other genome builds must be lifted over first

## Evidence

- [other] Use BedTools intersect with the -wo (write overlap) option to obtain, for every eccDNA–gene pair, the size of the overlapping genomic interval.: [evidence span withheld — anchor status: not_found]
- [abstract] Total eccDNA genes annotated: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [readme] README of eccDNAdb source code: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [other] GENCODE annotation source used: [evidence span withheld — anchor status: not_found]
