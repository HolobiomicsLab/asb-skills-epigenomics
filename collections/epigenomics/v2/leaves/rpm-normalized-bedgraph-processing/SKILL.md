---
name: rpm-normalized-bedgraph-processing
description: Use when starting CIPHER's enhancer-prediction or element-feature computation
  and you have ENCODE ChIP-seq/DNase-seq data that must be converted to RPM-normalized
  bedGraph signal tracks before genomic elements are classified.
when_to_use_negative:
- You need only a subset of CIPHER's tools rather than a whole workflow — the article
  notes 'CIPHER currently only runs entire workflows, not subsets of tools'
- Your data are raw reads and you expect the enhancer model itself to handle mapping/normalization
  — RPM-normalized bedGraph signal must be produced by the upstream CIPHER pre-processing
  steps first
- You mix single-ended and pair-ended data in the same config file, which CIPHER does
  not handle
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: CIPHER
  role: Consumes RPM-normalized bedGraph signal files to compute chromatin-signature
    features and run the predictEnhancers random-forest classifier
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language that launches CIPHER's analysis mode ('--function predictEnhancers')
    over the bedGraph inputs
  repo: https://www.nextflow.io/
- name: randomForest
  role: R package supplying the random-forest model that classifies 200-bp bins using
    the bedGraph-derived average coverage features
- name: MACS2
  role: Calls DHS peaks used to validate predicted enhancers by >=1 bp overlap at
    q<0.01
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:52ef017644957aef5f0d821df5fdb35a0251898d9ea10b87c3a435b78a935474
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - MACS2
  - Nextflow
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# rpm-normalized-bedgraph-processing

## Summary

A CIPHER workflow pattern in which RPM-normalized bedGraph signal files of DNase-seq and ChIP-seq (H3K4me1, H3K4me3, H3K27ac) data are consumed as standardized coverage inputs for the random-forest enhancer predictor ('--function predictEnhancers') and for computing chromatin-signature features over genomic elements.

## When to use

Use when starting CIPHER's enhancer-prediction or element-feature computation and you have ENCODE ChIP-seq/DNase-seq data that must be converted to RPM-normalized bedGraph signal tracks before genomic elements are classified.

## When NOT to use

- You need only a subset of CIPHER's tools rather than a whole workflow — the article notes 'CIPHER currently only runs entire workflows, not subsets of tools'
- Your data are raw reads and you expect the enhancer model itself to handle mapping/normalization — RPM-normalized bedGraph signal must be produced by the upstream CIPHER pre-processing steps first
- You mix single-ended and pair-ended data in the same config file, which CIPHER does not handle

## Inputs

- ChIP-seq FASTQ or processed signal data for H3K4me1, H3K4me3, H3K27ac (e.g. ENCODE HCT116/HeLa)
- DNase-seq data (e.g. ENCODE)
- RPM-normalized bedGraph signal files per chromatin marker and cell line
- Reference genome FASTA and Ensembl/Gencode GTF files
- CIPHER config file (tab-separated 5-column single-end or 6-column pair-end)

## Outputs

- RPM-normalized bedGraph visualization tracks per dataset
- Classified 200-bp genome bins (enhancer vs non-enhancer)
- Merged predicted enhancer elements (active and primed)
- Per-cell-line table of enhancer coordinates with DHS-overlap validation status (q<0.01)

## How to apply

Process raw ChIP-seq (H3K4me1, H3K4me3, H3K27ac) and DNase-seq reads through CIPHER's pre-processing steps (read trimming, mapping and quality filtering) so that coverage is emitted as RPM-normalized bedGraph files per marker and cell line. Supply these bedGraphs to CIPHER in analysis mode via Nextflow ('--function predictEnhancers'), which splits the reference genome into 200-bp bins and computes model features as the average coverage signal of each marker within a set distance along each genomic element. These RPM-normalized signals feed the trained randomForest R model (~93% accuracy on held-out enhancers vs promoters); predicted enhancer bins are then merged and intersected with MACS2 DHS peaks (q<0.01, >=1 bp overlap) to label validated enhancers.

## Related tools

- **CIPHER** (Consumes RPM-normalized bedGraph signal files to compute chromatin-signature features and run the predictEnhancers random-forest classifier) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language that launches CIPHER's analysis mode ('--function predictEnhancers') over the bedGraph inputs) — https://www.nextflow.io/
- **randomForest** (R package supplying the random-forest model that classifies 200-bp bins using the bedGraph-derived average coverage features)
- **MACS2** (Calls DHS peaks used to validate predicted enhancers by >=1 bp overlap at q<0.01)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --function predictEnhancers --config config.tsv --fa hg38.fa --gtf gencode.gtf
```

## Evaluation signals

- Confusion-matrix accuracy on the held-out one-third test set (caret package in R) reaches slightly above 93%
- Every emitted 'validated' enhancer overlaps a MACS2 DHS peak by >=1 bp at q<0.01; no enhancer without such overlap is labeled validated
- Predicted enhancers reproduce the expected DNase-sensitivity pattern and chromatin signatures (H3K4me1/H3K27ac high, H3K4me3 low) and eRNA transcriptional activity in each cell line
- Enhancer bins within 1 bp of each other are merged into single continuous enhancer elements before validation
- Signal values used as features correspond to RPM-normalized coverage averaged along each genomic element, not raw read counts

## Limitations

- CIPHER cannot mix single-ended and pair-ended data in the same config file ('DO NOT MIX AND MATCH... CIPHER DOES NOT HANDLE THIS USE-CASE YET.')
- On a local machine the workflow can run out of memory; the README instructs setting the -qs flag to limit parallelized processes
- RPM-normalized bedGraph processing requires completed mapping/QC runs, so enhancer prediction inherits any upstream alignment or quality-control failures

## Evidence

- [methods] Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
- [methods] Compute model features as the average coverage signal of each chromatin marker: [evidence span withheld — anchor status: not_found]
- [methods] CIPHER enhancer prediction-model accuracy achieved slightly above 93%: "CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%"
- [readme] README: CIPHER is a data processing workflow platform for next generation sequencing data including ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq and GRO-seq.: "CIPHER is a data processing workflow platform for next generation sequencing data including ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq and GRO-seq."
