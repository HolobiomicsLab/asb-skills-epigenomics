---
name: genomic-enhancer-prediction
description: Use when the goal is to classify genomic regions as enhancers from matched
  ChIP-seq (H3K4me1, H3K4me3, H3K27ac) and DNase-seq datasets, e.g. reproducing enhancer
  catalogs in human cell lines like HCT116 or HeLa.
when_to_use_negative:
- When only one or two histone marks are available without DNase-seq — the model requires
  the combination of H3K4me3, H3K4me1, H3K27ac ChIP-seq and DNase-seq data
- When the task is generic NGS pre-processing (trimming, mapping, QC) rather than
  enhancer/regulatory-element classification
edam_operation: http://edamontology.org/operation_2454
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0797
tools:
- name: CIPHER
  role: Runs the enhancer prediction workflow ('--function predictEnhancers') integrating
    ChIP-seq and DNase-seq data with its random-forest model
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language used to launch and orchestrate the CIPHER enhancer prediction
    pipeline
- name: Docker
  role: Container-based virtualization technology used by CIPHER for reproducible
    execution of pipeline components
- name: Singularity
  role: Container technology (per README) used to create the cipher.img image and
    execute the workflow reproducibly
- name: randomForest
  role: R package providing the trained random-forest classifier applied to each 200-bp
    bin
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:4556e6d291c9e0727cf91329f59b0df8f195dcb20e4387c0ca71a4c9bbc2b1c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - Nextflow
  - Singularity
  - randomForest
  edam_operation: http://edamontology.org/operation_2454
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# genomic-enhancer-prediction

## Summary

Predict active and primed enhancer elements genome-wide using CIPHER's built-in random-forest model, which integrates ChIP-seq marks (H3K4me1, H3K4me3, H3K27ac) with DNase-seq in 200-bp genomic bins, as validated in HCT116 and HeLa cell lines.

## When to use

Use when the goal is to classify genomic regions as enhancers from matched ChIP-seq (H3K4me1, H3K4me3, H3K27ac) and DNase-seq datasets, e.g. reproducing enhancer catalogs in human cell lines like HCT116 or HeLa.

## When NOT to use

- When only one or two histone marks are available without DNase-seq — the model requires the combination of H3K4me3, H3K4me1, H3K27ac ChIP-seq and DNase-seq data
- When the task is generic NGS pre-processing (trimming, mapping, QC) rather than enhancer/regulatory-element classification

## Inputs

- RPM-normalized bedGraph files of DNase-seq
- RPM-normalized bedGraph files of ChIP-seq for H3K4me1, H3K4me3, and H3K27ac
- Reference genome (e.g. UCSC hg19/hg38) for binning into 200-bp intervals
- MACS2-called DHS peak list (q<0.01) for validation

## Outputs

- Per-cell-line table of predicted enhancer coordinates with DHS-overlap validation status
- Counts of 'active' and 'primed' enhancers (e.g. 18,877 active / 11,460 primed in HCT116; 38,045 active / 10,600 primed in HeLa)

## How to apply

Launch CIPHER in analysis mode ('--function predictEnhancers') via Nextflow, supplying RPM-normalized bedGraph files of DNase-seq and ChIP-seq (H3K4me1, H3K4me3, H3K27ac) for each cell line. Split the reference genome into 200-bp bins and apply the trained random-forest model (randomForest package in R) to label each bin 'enhancer' or 'non-enhancer'. Merge enhancer bins lying within 1 bp of each other into continuous enhancer elements. Validate predictions by intersecting them with MACS2-called DNase hypersensitive sites (DHS) at q<0.01, marking an enhancer 'validated' on any >=1 bp overlap; the model accuracy benchmark is slightly above 93%. Compile a per-cell-line table of enhancer coordinates with DHS overlap status.

## Related tools

- **CIPHER** (Runs the enhancer prediction workflow ('--function predictEnhancers') integrating ChIP-seq and DNase-seq data with its random-forest model) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language used to launch and orchestrate the CIPHER enhancer prediction pipeline)
- **Docker** (Container-based virtualization technology used by CIPHER for reproducible execution of pipeline components)
- **Singularity** (Container technology (per README) used to create the cipher.img image and execute the workflow reproducibly)
- **randomForest** (R package providing the trained random-forest classifier applied to each 200-bp bin)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --function predictEnhancers --config CONFIG --fa hg19.fa --gtf genes.gtf
```

## Evaluation signals

- Model accuracy on held-out data should be slightly above 93%, matching the paper's benchmark
- Only enhancers meeting the >=1 bp DHS-overlap criterion at q<0.01 should be emitted as validated
- Predicted enhancer counts in HCT116 (18,877 active / 11,460 primed) and HeLa (38,045 active / 10,600 primed) serve as reference magnitudes for comparable ENCODE inputs
- Predictions should show the expected DNase-sensitivity pattern, chromatin signatures, and eRNA transcriptional activity consistent with known enhancer features
- Neighboring enhancer bins within 1 bp must be merged into single continuous enhancer elements before reporting

## Limitations

- CIPHER currently only runs entire workflows, not subsets of tools, so enhancer prediction cannot be extracted as a standalone tool step
- The README notes CIPHER does not yet handle mixing single-ended and pair-ended data in the same CONFIG file
- Too much parallelization (missing -qs flag on non-cluster runs) can exhaust memory and abruptly end the workflow
