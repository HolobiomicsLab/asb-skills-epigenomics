---
name: random-forest-classifier-application
description: Use when genomic bins must be classified as active/primed enhancers from
  ChIP-seq histone marks and DNase-seq accessibility data, e.g. reproducing enhancer
  discovery in human cell lines like HCT116 or HeLa.
when_to_use_negative:
- Input data lacks the required ChIP-seq marks (H3K4me1, H3K4me3, H3K27ac) or DNase-seq
  — the model cannot score bins without these signals
- Analysis goal is general peak calling or motif identification rather than enhancer
  element prediction (use CIPHER's standard ChIP-seq or DNase-seq pipelines instead)
- Non-model organism genomes without suitable ENCODE-derived enhancer training data
  for the random-forest model
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: CIPHER
  role: Workflow platform providing the '--function predictEnhancers' analysis-mode
    function that wraps the random-forest enhancer prediction model
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language used to launch and orchestrate the CIPHER enhancer prediction
    run
  repo: https://www.nextflow.io/
- name: Docker
  role: Containerization technology used by CIPHER to package and reproduce the prediction
    software environment (README describes Singularity as the shipped container option)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:c19728d7a5fed98679d8169a1e20175735846532c0854ade00df7b352b5a2100
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - Nextflow
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# random-forest-classifier-application

## Summary

Applies CIPHER's trained random-forest enhancer prediction model (R randomForest package) to classify 200-bp genomic bins as enhancer or non-enhancer, using ChIP-seq (H3K4me1, H3K4me3, H3K27ac) and DNase-seq signals, achieving ~93% accuracy against ENCODE enhancer training data.

## When to use

Use when genomic bins must be classified as active/primed enhancers from ChIP-seq histone marks and DNase-seq accessibility data, e.g. reproducing enhancer discovery in human cell lines like HCT116 or HeLa.

## When NOT to use

- Input data lacks the required ChIP-seq marks (H3K4me1, H3K4me3, H3K27ac) or DNase-seq — the model cannot score bins without these signals
- Analysis goal is general peak calling or motif identification rather than enhancer element prediction (use CIPHER's standard ChIP-seq or DNase-seq pipelines instead)
- Non-model organism genomes without suitable ENCODE-derived enhancer training data for the random-forest model

## Inputs

- RPM-normalized bedGraph files of DNase-seq signal per cell line
- RPM-normalized bedGraph files of ChIP-seq signal for H3K4me1, H3K4me3, and H3K27ac
- Reference genome FASTA (e.g. UCSC hg19/hg38) for 200-bp binning
- CIPHER config file listing samples (tab-separated MergeID/SampleID/paths/InputID/Mark)

## Outputs

- Per-bin 'enhancer' vs 'non-enhancer' classification across the genome
- Merged continuous enhancer elements (bins within 1 bp merged)
- Per-cell-line table of predicted enhancer coordinates with active/primed labels and DHS-overlap validation status

## How to apply

Launch CIPHER in analysis mode with '--function predictEnhancers' via Nextflow, supplying RPM-normalized bedGraph files of DNase-seq and ChIP-seq (H3K4me1, H3K4me3, H3K27ac) for each cell line. The genome is split into 200-bp bins, and the trained random-forest model (randomForest package in R) classifies each bin as 'enhancer' or 'non-enhancer' based on features learned from an ENCODE-derived enhancer training set. Adjacent enhancer bins lying within 1 bp of each other are merged into single continuous enhancer elements. Predictions are then labeled 'active' or 'primed' according to the chromatin mark combinations. Validate by intersecting predicted enhancers with MACS2-called DNase-seq DHS peaks (q<0.01), counting an enhancer as validated on any >=1 bp overlap, consistent with the model's reported ~93% accuracy benchmark.

## Related tools

- **CIPHER** (Workflow platform providing the '--function predictEnhancers' analysis-mode function that wraps the random-forest enhancer prediction model) — https://github.com/c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language used to launch and orchestrate the CIPHER enhancer prediction run) — https://www.nextflow.io/
- **Docker** (Containerization technology used by CIPHER to package and reproduce the prediction software environment (README describes Singularity as the shipped container option))

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --config config.txt --fa hg19.fa --gtf gencode.gtf --function predictEnhancers
```

## Evaluation signals

- Model accuracy on held-out ENCODE enhancer/non-enhancer bins should reach slightly above 93%, as reported in the paper's methods
- Only enhancers with >=1 bp overlap to MACS2 DHS peaks at q<0.01 should be emitted as 'validated'
- Reproduced cell-line counts should be of the expected magnitude: 18,877 active / 11,460 primed enhancers in HCT116 and 38,045 active / 10,600 primed in HeLa
- Predicted enhancers should show the expected DNase-sensitivity pattern, chromatin signatures, and eRNA transcriptional activity consistent with known enhancer features
- All enhancer bins within 1 bp of each other must be merged into single continuous enhancer elements before reporting coordinates

## Limitations

- CIPHER currently only runs entire workflows, not subsets of tools, so predictEnhancers cannot be run standalone without the Nextflow framework
- Prediction quality depends on RPM-normalized bedGraph inputs; improperly normalized or missing ChIP-seq/DNase-seq tracks will degrade the random-forest classification
- The model was trained on ENCODE-derived human enhancer sets, so transfer to non-human or poorly annotated genomes is untested in the paper

## Evidence

- [methods] Enhancer prediction model accuracy slightly above 93%: "CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%"
- [methods] predictEnhancers function definition: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
- [methods] ENCODE-derived enhancer training class: "one class containing a previously determined set of enhancer elements from the Encyclopedia of DNA Elements (ENCODE) project [40]"
- [full_text] Workflow and validation criteria from source card: [evidence span withheld — anchor status: not_found]
- [abstract] CIPHER analysis mode scope: "CIPHER contains an "analysis" mode that completes complex bioinformatics tasks such as enhancer discovery and provides functions to integrate various datasets together."
