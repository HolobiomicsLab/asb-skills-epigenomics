---
name: rnaseq-design-matrix-construction
description: Use when an RNA-seq count matrix covers more than two conditions, multiple
  factors, batch effects, or a time course, and the default two-group DEG schema cannot
  express the contrasts of interest.
when_to_use_negative:
- The study is a simple two-type design (e.g. WT versus KO) that CIPHER's built-in
  RNA-seq DEG pipeline already supports
- Input is raw FASTQ rather than a quantified count matrix — mapping and featureCounts
  must run first
- Count data have not been QC'd/normalized (FastQC/QoRTs/MultiQC, RUVSeq) and unwanted
  variation is uncharacterized
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Constructs the multi-factor/time-series design matrix (e.g. ~ time + condition
    + time:condition) and performs negative-binomial differential expression testing
- name: edgeR
  role: Fits a parallel GLM design for the same contrasts as a cross-check against
    DESeq2
- name: RUVSeq
  role: Assesses and adjusts unwanted variation in the count matrix before design-matrix
    fitting
- name: featureCounts
  role: Produces the gene-level counts the design matrix is built on
- name: CIPHER
  role: Upstream Nextflow workflow that aligns (STAR/BBMap) and quantifies reads;
    its two-condition DEG limitation is what this skill extends
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language underlying CIPHER's pre-processing that feeds the count
    matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:1e3070c66ec9a2ff77185499d2a50d3b6dd26a370369ff82a805deaf7254bb2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - DESeq2
  - Nextflow
  - QoRTs
  - RUVSeq
  - STAR
  - edgeR
  - featureCounts
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# rnaseq-design-matrix-construction

## Summary

Constructs multi-factor and time-series design matrices for RNA-seq differential expression analysis (e.g. ~ condition + batch or ~ time + condition + time:condition) in DESeq2 and edgeR, addressing CIPHER's documented restriction to two-type designs such as WT versus KO.

## When to use

Use when an RNA-seq count matrix covers more than two conditions, multiple factors, batch effects, or a time course, and the default two-group DEG schema cannot express the contrasts of interest.

## When NOT to use

- The study is a simple two-type design (e.g. WT versus KO) that CIPHER's built-in RNA-seq DEG pipeline already supports
- Input is raw FASTQ rather than a quantified count matrix — mapping and featureCounts must run first
- Count data have not been QC'd/normalized (FastQC/QoRTs/MultiQC, RUVSeq) and unwanted variation is uncharacterized

## Inputs

- Gene-level count matrix from featureCounts quantification (post-RUVSeq normalization)
- Tab-separated CONFIG file (MergeID, ID, Path, ...) mapping samples to FASTQ files
- Experimental sample metadata / expInfo-style file with sample and condition (plus time, batch where relevant)
- GTF annotation used for quantification

## Outputs

- DESeq2 and edgeR design matrices encoding multi-factor or time-series formulas
- Fitted model objects for tested contrasts
- Combined DEG results table with per-gene log2 fold-change, p-value, and FDR per contrast, plus design-formula, factor-level, and tested-coefficient metadata columns

## How to apply

First normalize the gene-level count matrix (produced by STAR/BBMap mapping and featureCounts quantification in CIPHER) with RUVSeq to assess unwanted variation before modeling. Then build a sample-sheet (expInfo-style: sample, condition) covering all factor levels and construct a DESeq2 design matrix such as ~ condition + batch for multi-factor studies or ~ time + condition + time:condition for time-series interaction testing; in parallel, fit an edgeR GLM design over the same factors. Fit the models and test contrasts that a two-condition schema cannot express, recording per-gene log2 fold-change, p-value, and FDR per contrast. Augment the DEG results table with model-design metadata columns (design formula, factor levels, tested coefficient) so the lifted two-condition restriction is auditable. Rationale: CIPHER's discussion explicitly states its RNA-seq pipeline is optimized for typical two-type experimental designs, so multi-design capability must be supplied externally.

## Related tools

- **DESeq2** (Constructs the multi-factor/time-series design matrix (e.g. ~ time + condition + time:condition) and performs negative-binomial differential expression testing)
- **edgeR** (Fits a parallel GLM design for the same contrasts as a cross-check against DESeq2)
- **RUVSeq** (Assesses and adjusts unwanted variation in the count matrix before design-matrix fitting)
- **featureCounts** (Produces the gene-level counts the design matrix is built on)
- **CIPHER** (Upstream Nextflow workflow that aligns (STAR/BBMap) and quantifies reads; its two-condition DEG limitation is what this skill extends) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language underlying CIPHER's pre-processing that feeds the count matrix)

## Examples

```
dds <- DESeqDataSetFromMatrix(countData=counts, colData=expInfo, design=~time+condition+time:condition); res <- results(dds, contrast=list("timecondition"))
```

## Evaluation signals

- Output DEG table exists and contains design-metadata columns (design formula, factor levels, tested coefficient) plus FDR for all quantified genes
- Recorded design formula supports more than two factor levels (e.g. three conditions or a time-course), confirming the two-condition restriction is lifted
- DESeq2 and edgeR GLM fits both run on the same sample sheet without errors and produce non-empty results for each specified contrast
- Per-contrast results include log2 fold-change, p-value, and FDR for every gene in the featureCounts count matrix
- RUVSeq normalization diagnostics show unwanted-variation factors were estimated before model fitting

## Limitations

- CIPHER's RNA-seq pipeline is largely optimized only for typical two-type experimental designs (e.g. WT versus KO), so multi-design and time-series DEG analysis must be run outside its built-in schema
- CIPHER does not handle mixed single-end and pair-ended data in the same CONFIG file, constraining which samples can be combined into one design
- CIPHER currently only runs entire workflows, not subsets of tools, so the design-matrix step cannot be invoked modularly within CIPHER

## Evidence

- [abstract] differential gene expression analysis for RNA-seq: "narrow and broad peak calling, peak annotation, and motif identification for ChIP-seq, differential gene expression analysis for RNA-seq"
- [discussion] two-type experimental designs limitation: "CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO)"
- [readme] CONFIG schema: [evidence span withheld — anchor status: external_doc]
- [intro] Nextflow basis: "By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept"
