---
name: rna-seq-expression-based-gene-stratification
description: Use when you need to partition a gene set into expression-based classes
  (e.g. three groups of 100 highly/moderately/silently expressed RefSeq genes) from
  RNA-seq data to relate them to ChIP-seq signal levels.
when_to_use_negative:
- When the analysis goal is continuous expression-vs-signal regression rather than
  comparing discrete expression classes via boxplots
- When the genome or annotation is non-RefSeq (SeqCode relies on RefSeq reference
  annotations), which would break consistency between the gene lists and the recoverChIPlevels
  input
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3169
tools:
- name: SeqCode
  role: Consumes the expression-stratified gene lists to compute normalized ChIP levels
    (avg/max/total) over gene bodies per class via recoverChIPlevels
  repo: https://github.com/eblancoga/seqcode
- name: R
  role: Renders the boxplots of per-class ChIP levels for each mark from the recovered
    signal values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-98889-7
    title: Productive visualization of high-throughput sequencing data using the SeqCode
      open portable platform
  source_content_hash: sha256:c51fe3fecb1c499ab65209036fc446fc21c881c6876650895f62c68ba2c8d4dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eblancoga/seqcode
  tools:
  - Bowtie
  - R
  - SAMtools
  - SeqCode
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# RNA-seq expression-based gene stratification

## Summary

Stratify RefSeq genes into three expression classes (highly, moderately, and silently expressed) by ranking genes on published mESC RNA-seq data (GSM1562339, Jacinto et al.), so that normalized ChIP-seq signal of chromatin marks can be compared across classes to test expression-chromatin correlations.

## When to use

Use when you need to partition a gene set into expression-based classes (e.g. three groups of 100 highly/moderately/silently expressed RefSeq genes) from RNA-seq data to relate them to ChIP-seq signal levels.

## When NOT to use

- When the analysis goal is continuous expression-vs-signal regression rather than comparing discrete expression classes via boxplots
- When the genome or annotation is non-RefSeq (SeqCode relies on RefSeq reference annotations), which would break consistency between the gene lists and the recoverChIPlevels input

## Inputs

- Published mESC RNA-seq data (GSM1562339, Jacinto et al.) from NCBI GEO
- RefSeq gene transcript annotation for the genome assembly (e.g. mm9 refGene file from UCSC)
- Chromosome sizes / chromInfo file for mm9

## Outputs

- Three ranked gene lists of 100 RefSeq genes each: highly, moderately, and silently expressed
- Gene-group region files used as input to SeqCode recoverChIPlevels for per-class ChIP signal computation

## How to apply

Retrieve the published mESC RNA-seq data (GSM1562339, Jacinto et al.) and rank RefSeq genes by expression level to select three groups of 100 genes: highly, moderately, and silently expressed. Keep the transcript catalog consistent with the downstream ChIP analysis: SeqCode relies on RefSeq reference annotations, so the same mm9 refGene annotation file used by recoverChIPlevels should define the gene bodies for each group. The stratification produces one gene list per expression class that serves as the region input for computing per-group normalized ChIP read counts (normalized by total reads in the BAM, i.e. sequencing depth). The rationale is to convert a continuous expression axis into discrete classes whose ChIP-level distributions (boxplots) can be compared qualitatively for correlation direction.

## Related tools

- **SeqCode** (Consumes the expression-stratified gene lists to compute normalized ChIP levels (avg/max/total) over gene bodies per class via recoverChIPlevels) — https://github.com/eblancoga/seqcode
- **R** (Renders the boxplots of per-class ChIP levels for each mark from the recovered signal values)

## Examples

```
bin/recoverChIPlevels -v mm9/chromInfo.txt mm9/refGene.txt H3K4me3.bam expression_groups.txt 5000
```

## Evaluation signals

- Each expression group contains exactly 100 RefSeq genes (high, moderate, silent classes)
- Gene identity and coordinates are consistent with the mm9 RefSeq annotation used by SeqCode commands
- Downstream boxplots reproduce the reported qualitative pattern: active marks (H3K4me3, H3K36me3, Ser5P, Ser2P) high on highly expressed genes and Polycomb marks (H3K27me3, Suz12, Jarid2, Ring1b, H2Aub) high on silent genes
- Correlation directions with expression match Fig. 6 qualitatively; no numeric threshold is specified in the article

## Limitations

- The article reports no numeric acceptance threshold — only qualitative concordance of correlation directions with Fig. 6 validates the output
- Stratification into only three discrete classes of 100 genes discards expression information for the remaining genes
- Results depend on the RNA-seq dataset used for ranking (here GSM1562339, Jacinto et al.); a different dataset could yield different class assignments

## Evidence

- [other] Download the published mESC RNA-seq data (GSM1562339, Jacinto et al.) and rank RefSeq genes to select three groups of 100 genes: highly, moderately, and silently expressed.: [evidence span withheld — anchor status: not_found]
- [methods] SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release of every organism that is available from this consortium.: "SeqCode relies on the RefSeq reference annotations and is able to deal with the genome and assembly release"
- [other] gene expression positively correlated with H3K4me3, H3K36me3, Ser5P, and Ser2P, and negatively correlated with H3K27me3, Suz12, Jarid2, Ring1b, and H2Aub.: [evidence span withheld — anchor status: not_found]
