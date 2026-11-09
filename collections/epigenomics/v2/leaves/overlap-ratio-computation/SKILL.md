---
name: overlap-ratio-computation
description: Use when genomic interval sets (e.g., eccDNA coordinates and GENCODE
  gene annotations) must be intersected and pairwise overlap fractions computed to
  decide which genes fall on which eccDNA elements.
when_to_use_negative:
- The input interval sets are already merged, annotated, or in a format where overlap
  sizes are already computed — recomputing Re/Rg would be redundant.
- The goal is eccDNA identification from raw WGS data rather than post-hoc gene annotation
  of an existing eccDNA catalog; use the AmpliconArchitect detection pipeline first.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: BedTools
  role: Performs interval intersection with -wo to output overlapping intervals and
    overlap sizes between the eccDNA catalog and gene annotation BED files
- name: GENCODE
  role: Source of the comprehensive GRCh37 (release 38) gene annotation converted
    to BED for intersecting against the eccDNA catalog
- name: AmpliconArchitect
  role: Upstream tool that identified the eccDNA elements whose chromosomal coordinates
    form the input catalog for overlap computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:b857b790d633de96308ef0870240566e0e5e4c4fb908d10a197c119efba478dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AmpliconArchitect
  - BedTools
  - GENCODE
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# overlap-ratio-computation

## Summary

Computes normalized reciprocal overlap ratios (Re = overlap/eccDNA length; Rg = overlap/gene length) between an eccDNA catalog and gene annotations using BedTools intersect -wo, to annotate eccDNA genes for eccDNAdb.

## When to use

Use when genomic interval sets (e.g., eccDNA coordinates and GENCODE gene annotations) must be intersected and pairwise overlap fractions computed to decide which genes fall on which eccDNA elements.

## When NOT to use

- The input interval sets are already merged, annotated, or in a format where overlap sizes are already computed — recomputing Re/Rg would be redundant.
- The goal is eccDNA identification from raw WGS data rather than post-hoc gene annotation of an existing eccDNA catalog; use the AmpliconArchitect detection pipeline first.

## Inputs

- BED interval file of eccDNA chromosomal coordinates (from the AmpliconArchitect-identified eccDNA catalog)
- GENCODE GRCh37 (release 38) comprehensive gene annotation converted to a compatible BED interval file

## Outputs

- eccdna_gene_annotation.tsv: tab-delimited table of gene, eccDNA, overlap size, Re, and Rg per eccDNA–gene pair

## How to apply

Convert the eccDNA catalog (chromosomal coordinates of identified eccDNA elements) and the GENCODE GRCh37 (release 38) comprehensive gene annotation into compatible BED interval files. Run BedTools intersect with the -wo (write overlap) option so that every eccDNA–gene pair is returned together with the size of its overlapping genomic interval. For each pair compute Re as the overlap size divided by the eccDNA interval size and Rg as the overlap size divided by the gene interval size; both must lie in [0, 1]. Aggregate per gene and write the gene-annotation table (gene, eccDNA, overlap size, Re, Rg) to eccdna_gene_annotation.tsv. Applying this workflow to the eccDNAdb catalog produced 54,901 annotated eccDNA genes, averaging about 7 genes per eccDNA.

## Related tools

- **BedTools** (Performs interval intersection with -wo to output overlapping intervals and overlap sizes between the eccDNA catalog and gene annotation BED files)
- **GENCODE** (Source of the comprehensive GRCh37 (release 38) gene annotation converted to BED for intersecting against the eccDNA catalog)
- **AmpliconArchitect** (Upstream tool that identified the eccDNA elements whose chromosomal coordinates form the input catalog for overlap computation)

## Examples

```
bedtools intersect -a eccdna_catalog.bed -b gencode_v38_genes.bed -wo > eccdna_gene_annotation.tsv
```

## Evaluation signals

- Output file eccdna_gene_annotation.tsv exists and is well-formed tab-delimited text with the required Re and Rg columns
- All Re and Rg values fall within the range [0, 1]
- Reproduce the article-scale result: ~54,901 annotated eccDNA genes and an average of ~7 genes per eccDNA when run on the full eccDNAdb catalog
- Every overlap size recorded is consistent with an actual intersection between the two BED interval sets (no overlap pairs without a positive overlap size)

## Limitations

- All eccDNAs in the database were identified by the AmpliconArchitect pipeline only, so the eccDNA input catalog inherits that pipeline's detection biases
- The annotation depends on GENCODE GRCh37 (release 38); coordinate-system mismatches between the catalog and gene annotation will yield spurious or missed overlaps
- No direct URL or access location for eccDNAdb was provided in the discussion section, limiting reproducibility checks from the article alone

## Evidence

- [abstract] A total of 54,901 eccDNA genes were annotated and included in the database as well: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [discussion] AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25].: "AmpliconArchitect is a robust and feasible tool for detecting eccDNA elements from WGS data [25]."
- [readme] The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes.: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
