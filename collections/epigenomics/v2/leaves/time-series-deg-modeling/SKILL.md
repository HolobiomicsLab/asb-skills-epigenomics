---
name: time-series-deg-modeling
description: Use when an RNA-seq study has three or more conditions, a time-course,
  or batch effects, so a two-type design matrix (as CIPHER's expInfo schema supports)
  cannot express the contrasts of interest.
when_to_use_negative:
- The experiment is a simple two-type design (e.g. WT versus KO) fully expressible
  in CIPHER's existing expInfo two-condition schema
- Input is already a gene count matrix and only CIPHER's upstream preprocessing (QC,
  STAR alignment, featureCounts) is needed
- De novo transcriptome assembly is the goal — the article explicitly lists this as
  a missing, future CIPHER capability
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CIPHER
  role: Baseline workflow platform whose two-condition RNA-seq DEG schema this skill
    extends; supplies QC, alignment (--aligner STAR), and quantification steps
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: DESeq2
  role: Fits multi-factor design matrices (e.g. ~ time + condition + time:condition)
    on RUVSeq-normalized counts to test contrasts CIPHER cannot express
- name: edgeR
  role: Parallel GLM-based design and contrast testing used alongside DESeq2
- name: RUVSeq
  role: Assessment and normalization of unwanted variation in the gene-level count
    matrix prior to model fitting
- name: featureCounts
  role: Gene-level count quantification from STAR alignments using the supplied GTF
    annotation
- name: STAR
  role: Splice-aware aligner selected through CIPHER's --aligner flag (default BBMap
    retained as fallback)
- name: Nextflow
  role: Workflow language underlying CIPHER; used to orchestrate the upstream preprocessing
    steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:4672b82d98b7b02894e287bd4ceadb8e5053095ddbd5418422990b8de70f1256
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# time-series-deg-modeling

## Summary

Extends CIPHER's RNA-seq differential expression workflow beyond its two-condition (e.g. WT versus KO) schema by constructing multi-factor DESeq2 and edgeR GLM design matrices (e.g. ~ time + condition + time:condition) over RUVSeq-normalized gene counts, enabling time-series and multi-factor DEG analysis.

## When to use

Use when an RNA-seq study has three or more conditions, a time-course, or batch effects, so a two-type design matrix (as CIPHER's expInfo schema supports) cannot express the contrasts of interest.

## When NOT to use

- The experiment is a simple two-type design (e.g. WT versus KO) fully expressible in CIPHER's existing expInfo two-condition schema
- Input is already a gene count matrix and only CIPHER's upstream preprocessing (QC, STAR alignment, featureCounts) is needed
- De novo transcriptome assembly is the goal — the article explicitly lists this as a missing, future CIPHER capability

## Inputs

- Raw FASTQ files from a multi-factor or time-course RNA-seq study (e.g. from GEO/SRA)
- Tab-separated CIPHER config file with columns MergeID, ID, Path
- Experimental config file (sample, condition) matching CIPHER's expInfo schema
- GTF gene annotation file (--gtf) and reference FASTA (--fa)
- RUVSeq-normalized gene-level count matrix from featureCounts

## Outputs

- Combined DEG results table with per-gene log2 fold-change, p-value, and FDR for each contrast
- Design-metadata columns: design formula, factor levels, and tested coefficient per contrast
- MultiQC aggregate HTML QC report from FastQC/QoRTs

## How to apply

Start from CIPHER's standard RNA-seq preprocessing: FastQC/QoRTs QC aggregated with MultiQC, STAR splice-aware alignment via CIPHER's --aligner flag (BBMap is the default fallback), gene-level quantification with featureCounts from the Subread suite using the supplied GTF, then RUVSeq normalization of the count matrix. Input samples via a tab-separated config file (MergeID, ID, Path) consistent with CIPHER's expInfo schema, plus an experimental config (sample, condition). Instead of CIPHER's built-in two-condition DEG step, construct a DESeq2 design matrix such as ~ condition + batch or ~ time + condition + time:condition and, in parallel, an edgeR GLM design; fit both models and test the contrasts CIPHER's schema cannot express. Augment the combined per-gene DEG table (log2 fold-change, p-value, FDR) with model-design metadata columns recording the design formula, factor levels, and tested coefficient.

## Related tools

- **CIPHER** (Baseline workflow platform whose two-condition RNA-seq DEG schema this skill extends; supplies QC, alignment (--aligner STAR), and quantification steps) — https://github.com/c-guzman/cipher-workflow-platform
- **DESeq2** (Fits multi-factor design matrices (e.g. ~ time + condition + time:condition) on RUVSeq-normalized counts to test contrasts CIPHER cannot express)
- **edgeR** (Parallel GLM-based design and contrast testing used alongside DESeq2)
- **RUVSeq** (Assessment and normalization of unwanted variation in the gene-level count matrix prior to model fitting)
- **featureCounts** (Gene-level count quantification from STAR alignments using the supplied GTF annotation)
- **STAR** (Splice-aware aligner selected through CIPHER's --aligner flag (default BBMap retained as fallback))
- **Nextflow** (Workflow language underlying CIPHER; used to orchestrate the upstream preprocessing steps)

## Examples

```
dds <- DESeqDataSetFromMatrix(counts, coldata, design = ~ time + condition + time:condition); dds <- DESeq(dds); res <- results(dds, contrast=list("conditionX.timeY"))
```

## Evaluation signals

- Output DEG table exists and contains design-metadata columns (design formula, factor levels, tested coefficient) plus FDR for all quantified genes
- The recorded design formula supports more than two factor levels, confirming the two-condition restriction of CIPHER's expInfo schema is lifted
- DESeq2 and edgeR GLM models were both fit and per-contrast log2 fold-change, p-value, and FDR are populated for every contrast tested
- Upstream artifacts are present: MultiQC HTML report, STAR alignment output, and a featureCounts gene-level count matrix normalized by RUVSeq

## Limitations

- CIPHER's current RNA-seq pipeline is largely optimized only for typical two-type experimental designs (e.g. WT versus KO), so multi-design and time-series DEG analysis is missing and must be performed externally with DESeq2/edgeR
- CIPHER does not handle mixed single-ended and pair-ended data in the same config file, per the README warning
- CIPHER only runs entire workflows, not subsets of tools, so the DEG-modeling step cannot be slotted into CIPHER directly
- When running without Singularity containers, all required software must be installed manually as listed in the cipher.nf script

## Evidence

- [discussion] CIPHER's current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO): "CIPHER’s current RNA-seq pipeline is largely optimized for typical two-type experimental designs (e.g. WT versus KO)"
- [abstract] differential gene expression analysis for RNA-seq: "differential gene expression analysis for RNA-seq"
- [readme] CIPHER run command from README: [evidence span withheld — anchor status: external_doc]
- [intro] Nextflow as the workflow engine: "By combining Nextflow [11, 12] - a powerful workflow language based on the Unix pipe concept"
