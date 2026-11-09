---
name: dnase-hypersensitive-site-overlap-validation
description: Use when predicted enhancer coordinates (e.g. from CIPHER's --function
  predictEnhancers on H3K4me1/H3K4me3/H3K27ac ChIP-seq plus DNase-seq) need an orthogonal
  validation against DNase hypersensitive sites.
when_to_use_negative:
- Input is already a curated set of experimentally validated enhancers rather than
  model-predicted elements needing orthogonal DHS support
- No DNase-seq dataset exists for the cell line of interest, since the >=1 bp DHS-overlap
  criterion cannot be evaluated
- The goal is de novo enhancer discovery without any validation step, not validation
  of existing predictions
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CIPHER
  role: Runs the predictEnhancers analysis-mode function that trains/applies the random-forest
    model on H3K4me1, H3K4me3, H3K27ac ChIP-seq and DNase-seq bedGraphs to produce
    enhancer coordinates
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language used to launch and orchestrate the CIPHER enhancer prediction
    pipeline
- name: MACS2
  role: Calls DNase-seq hypersensitive site peaks at q<0.01 used as the overlap-validation
    set
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:04e3a7d0e782ce07180cbb699d327b7dfb05dde801acb309c8461b1083dfdbad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - MACS2
  - Nextflow
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# dnase-hypersensitive-site-overlap-validation

## Summary

Validates predicted enhancer elements by intersecting them with MACS2-called DNase-seq hypersensitive sites (DHS) and labeling an enhancer 'validated' when a DNase peak overlaps it by at least 1 bp at q<0.01, mirroring CIPHER's enhancer validation across human cell lines.

## When to use

Use when predicted enhancer coordinates (e.g. from CIPHER's --function predictEnhancers on H3K4me1/H3K4me3/H3K27ac ChIP-seq plus DNase-seq) need an orthogonal validation against DNase hypersensitive sites.

## When NOT to use

- Input is already a curated set of experimentally validated enhancers rather than model-predicted elements needing orthogonal DHS support
- No DNase-seq dataset exists for the cell line of interest, since the >=1 bp DHS-overlap criterion cannot be evaluated
- The goal is de novo enhancer discovery without any validation step, not validation of existing predictions

## Inputs

- RPM-normalized bedGraph files of DNase-seq for each cell line
- RPM-normalized bedGraph files of ChIP-seq for H3K4me1, H3K4me3, and H3K27ac
- MACS2-called DNase-seq DHS peak list (q<0.01)
- Reference genome (e.g. UCSC hg19) for 200-bp binning

## Outputs

- Per-cell-line table of predicted enhancer genomic coordinates with DHS overlap status (validated / not validated)
- Counts of predicted enhancer classes per cell line (e.g. 18,877 active and 11,460 primed in HCT116; 38,045 active and 10,600 primed in HeLa)

## How to apply

Launch CIPHER in analysis mode with '--function predictEnhancers', supplying RPM-normalized bedGraph files of DNase-seq and ChIP-seq (H3K4me1, H3K4me3, H3K27ac) for each cell line. The genome is split into 200-bp bins and the trained random-forest model (randomForest package in R) classifies each bin as 'enhancer' or 'non-enhancer'; enhancer bins within 1 bp of each other are merged into continuous elements. Independently call DHS peaks with MACS2 at q<0.01, then intersect predicted enhancers with the DHS peaks. Label each enhancer 'validated' only when a DNase peak overlaps it by at least a single bp, and compile a per-cell-line table of enhancer coordinates with their DHS overlap status. The rationale is that active enhancers show DNase sensitivity, so >=1 bp DHS overlap at q<0.01 reproduces the paper's validated-enhancer definition, consistent with the model's ~93% accuracy benchmark.

## Related tools

- **CIPHER** (Runs the predictEnhancers analysis-mode function that trains/applies the random-forest model on H3K4me1, H3K4me3, H3K27ac ChIP-seq and DNase-seq bedGraphs to produce enhancer coordinates) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language used to launch and orchestrate the CIPHER enhancer prediction pipeline)
- **MACS2** (Calls DNase-seq hypersensitive site peaks at q<0.01 used as the overlap-validation set)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --function predictEnhancers --config CONFIG
```

## Evaluation signals

- Only enhancers with >=1 bp overlap to a MACS2 DHS peak at q<0.01 are emitted as validated; any enhancer lacking overlap must be excluded
- Enhancer bins are 200-bp genomic bins merged when within 1 bp of each other, so final elements should be contiguous intervals with no <1 bp gaps
- Predicted enhancers show the expected DNase-sensitivity pattern, chromatin signatures, and eRNA transcriptional activity consistent with known enhancer features
- Model accuracy benchmark is slightly above 93%; predictions falling far below this expectation suggest misconfigured inputs (e.g. non-RPM-normalized bedGraphs)
- Reproduced cell-line counts should be in the range reported by the paper (18,877 active / 11,460 primed in HCT116; 38,045 active / 10,600 primed in HeLa)

## Limitations

- The random-forest model was trained on ENCODE-defined enhancer elements; performance on cell types or marks outside ENCODE training data is not established
- A >=1 bp overlap is a permissive validation criterion; it does not quantify enhancer activity strength
- CIPHER currently only runs entire workflows, not subsets of tools, so running only the enhancer validation portion requires running the full predictEnhancers pipeline

## Evidence

- [methods] Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
- [abstract] DNase hypersensitive site mapping, site annotation and motif identification for DNase-seq: "DNase hypersensitive site mapping, site annotation and motif identification for DNase-seq"
- [methods] CIPHER's enhancer prediction-model accuracy achieved slightly above 93%: "CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%"
- [results] We downloaded ChIP-seq (H3K4me1) from the ENCODE project in human colonic cancer cells (HCT116): "We downloaded ChIP-seq (H3K4me1) from the ENCODE project in human colonic cancer cells (HCT116)"
- [abstract] CIPHER contains an "analysis" mode that completes complex bioinformatics tasks such as enhancer discovery: "CIPHER contains an "analysis" mode that completes complex bioinformatics tasks such as enhancer discovery"
