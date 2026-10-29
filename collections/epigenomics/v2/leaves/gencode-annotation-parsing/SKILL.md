---
name: gencode-annotation-parsing
description: Use when a genomic interval catalog (e.g., eccDNAs from AmpliconArchitect)
  must be intersected with GENCODE gene annotations to assign overlapping genes, with
  per-pair overlap ratios quantifying reciprocal coverage.
when_to_use_negative:
- The goal is identification of eccDNA elements from WGS reads rather than annotation
  of an already-built eccDNA catalog (use the AmpliconArchitect pipeline instead).
- Input coordinates are not on the GENCODE annotation's reference build (GRCh37/release
  38) — cross-build interval intersect would yield spurious overlaps without prior
  liftover.
- Analysis needs gene expression or clinical context rather than interval overlap
  annotation (that integration is a separate eccDNAdb step using TCGA/CCLE data).
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BedTools
  role: Intersects eccDNA and gene BED intervals with -wo to report per-pair overlap
    sizes used to compute Re and Rg
- name: GENCODE
  role: Source of the GRCh37 (release 38) comprehensive gene annotation converted
    to BED for intersecting
- name: AmpliconArchitect
  role: Upstream tool that produced the eccDNA catalog from WGS data that this skill
    annotates against genes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41388-022-02286-x
    title: 'eccDNAdb: a database of extrachromosomal circular DNA profiles in human
      cancers'
  source_content_hash: sha256:9d31893a1ef3e8f6392b2216a431cdb85fd111696729160e4d337f98f052fbc3
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
yes_rate: 0.875
n_claims: 8
---

# gencode-annotation-parsing

## Summary

Convert GENCODE comprehensive gene annotation (GRCh37/release 38 GTF) and an eccDNA catalog into compatible BED interval files, then annotate eccDNA genes via BedTools intersect overlap ratios (Re, Rg). This is the gene-annotation step of eccDNAdb that produced 54,901 annotated eccDNA genes.

## When to use

Use when a genomic interval catalog (e.g., eccDNAs from AmpliconArchitect) must be intersected with GENCODE gene annotations to assign overlapping genes, with per-pair overlap ratios quantifying reciprocal coverage.

## When NOT to use

- The goal is identification of eccDNA elements from WGS reads rather than annotation of an already-built eccDNA catalog (use the AmpliconArchitect pipeline instead).
- Input coordinates are not on the GENCODE annotation's reference build (GRCh37/release 38) — cross-build interval intersect would yield spurious overlaps without prior liftover.
- Analysis needs gene expression or clinical context rather than interval overlap annotation (that integration is a separate eccDNAdb step using TCGA/CCLE data).

## Inputs

- eccDNA catalog with chromosomal coordinates (identified e.g. by AmpliconArchitect from WGS data)
- GENCODE GRCh37 (release 38) comprehensive gene annotation (GTF/GFF3 converted to BED)

## Outputs

- eccdna_gene_annotation.tsv: tab-delimited table of gene, eccDNA, overlap size, Re (overlap/eccDNA size), and Rg (overlap/gene size)
- Annotated eccDNA gene set (54,901 eccDNA genes in eccDNAdb)

## How to apply

Load the eccDNA catalog (chromosomal coordinates of identified eccDNA elements) and convert both it and the GENCODE GRCh37 (release 38) comprehensive gene annotation into compatible BED interval files. Run BedTools intersect with the -wo (write overlap) option so every eccDNA–gene pair reports the size of the overlapping genomic interval. For each pair, compute Re as the overlap divided by the eccDNA interval size and Rg as the overlap divided by the gene interval size, then aggregate per gene and write the annotation table (gene, eccDNA, overlap size, Re, Rg) to eccdna_gene_annotation.tsv. In eccDNAdb this step annotated 54,901 eccDNA genes, with counts varying among eccDNAs and an average of 7 genes per eccDNA.

## Related tools

- **BedTools** (Intersects eccDNA and gene BED intervals with -wo to report per-pair overlap sizes used to compute Re and Rg)
- **GENCODE** (Source of the GRCh37 (release 38) comprehensive gene annotation converted to BED for intersecting)
- **AmpliconArchitect** (Upstream tool that produced the eccDNA catalog from WGS data that this skill annotates against genes)

## Examples

```
bedtools intersect -wo -a eccdna_catalog.bed -b gencode_grch38_genes.bed > eccdna_gene_annotation.tsv
```

## Evaluation signals

- Output table eccdna_gene_annotation.tsv exists and is well-formed tab-delimited text with the required Re and Rg columns.
- All Re and Rg values fall within the range [0, 1].
- The annotated gene count matches the eccDNAdb result scale (54,901 eccDNA genes across the 1270-eccDNA catalog).
- Average genes per eccDNA is consistent with the reported average of 7 genes per eccDNA.
- Gene coordinates derive from GENCODE GRCh37 release 38 and share compatible chromosome naming with the eccDNA BED catalog.

## Limitations

- All eccDNAs in eccDNAdb were identified by the AmpliconArchitect pipeline only, so the annotated gene set inherits that identification bias.
- Database coverage is limited to human cancers; more species and diseases were planned for future updates.
- Overlap ratios alone do not resolve partial overlaps — both Re and Rg must be inspected to judge whether a gene is genuinely on an eccDNA.

## Evidence

- [methods] Use BedTools intersect with the -wo (write overlap) option to obtain, for every eccDNA–gene pair, the size of the overlapping genomic interval.: [evidence span withheld — anchor status: not_found]
- [abstract] 54,901 eccDNA genes annotated: "A total number of 54,901 eccDNA genes were annotated and included in the database as well"
- [readme] README description of the repo: "The source code for the identification of eccDNAs from whole genome sequencing data of human cancers and for the annotation of eccDNA-derived genes."
- [discussion] TCGA/CCLE gene annotation context: "EccDNA genes were identified and annotated with expression data from TCGA and CCLE databases."
