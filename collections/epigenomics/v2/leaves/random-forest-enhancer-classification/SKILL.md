---
name: random-forest-enhancer-classification
description: Use when labeled enhancer and promoter (non-enhancer) genomic element
  sets are available with ChIP-seq and DNase-seq signal tracks (e.g. RPM-normalized
  bedGraph) and the goal is to train, validate, or apply a supervised enhancer-prediction
  model.
when_to_use_negative:
- The input lacks labeled enhancer/non-enhancer training classes — the model is supervised
  and cannot be trained on unlabeled genomic elements
- Only one mark (e.g. H3K27ac alone) is available — the model relies on a combination
  of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data
- The goal is NGS preprocessing (read mapping, trimming, QC) rather than enhancer
  classification — use CIPHER's standard pre-processing workflows instead
tools:
- name: CIPHER
  role: Workflow platform exposing the enhancer-prediction task via its analysis mode
    ('--function predictEnhancers'), combining ChIP-seq (H3K4me3, H3K4me1, H3K27Ac)
    and DNase-seq data
  repo: github.com__c-guzman__cipher-workflow-platform
- name: randomForest
  role: R package used to train the RFT model, generating binary classification decision
    trees until enhancers are separated from non-enhancers
- name: caret
  role: R package used to compute the confusion matrix and validate the trained model
    on held-out test classes
- name: Nextflow
  role: Workflow language that orchestrates the CIPHER enhancer-prediction run
  repo: https://www.nextflow.io/
- name: Docker
  role: Container-based virtualization technology used by CIPHER to package and reproduce
    the analysis software
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:3d7e861bcacb2f52945a5f144f0b6c4969ddac36c9d176b607a6b58982c196a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CIPHER
  - Docker
  - Nextflow
  - caret
  - randomForest
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-forest-enhancer-classification

## Summary

Train and validate a random-forest (RFT) binary classifier that separates transcriptional enhancers from promoter regions using average RPM-normalized coverage signals of DNase-seq DHS plus H3K4me1, H3K4me3, and H3K27ac over genomic elements, as implemented in CIPHER's predictEnhancers function.

## When to use

Use when labeled enhancer and promoter (non-enhancer) genomic element sets are available with ChIP-seq and DNase-seq signal tracks (e.g. RPM-normalized bedGraph) and the goal is to train, validate, or apply a supervised enhancer-prediction model.

## When NOT to use

- The input lacks labeled enhancer/non-enhancer training classes — the model is supervised and cannot be trained on unlabeled genomic elements
- Only one mark (e.g. H3K27ac alone) is available — the model relies on a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data
- The goal is NGS preprocessing (read mapping, trimming, QC) rather than enhancer classification — use CIPHER's standard pre-processing workflows instead

## Inputs

- Previously determined enhancer elements from the ENCODE project (positives)
- Promoter regions spanning -1/+1 kb around TSS of known genes (negatives)
- RPM-normalized bedGraph signal files for DNase-seq DHS and ChIP-seq marks H3K4me1, H3K4me3, H3K27ac
- Class labels (enhancer vs non-enhancer) for training and held-out test elements

## Outputs

- Trained random-forest enhancer-prediction model (saved RFT model object)
- Confusion matrix from the caret package on the held-out test set
- Metrics record containing the confusion-matrix-derived classification accuracy

## How to apply

Load positives as a previously determined set of enhancer elements from the ENCODE project and negatives as an equal number of promoter regions spanning -1/+1 kb from the transcription start site of known genes. Compute features as the average coverage signal of each marker (DNase-seq DHS plus H3K4me1, H3K4me3, and H3K27ac) within a set distance along each element, from RPM-normalized bedGraph signal files. Hold out one third of the classes for testing, then train the RFT model in R (version 3.3.1) with the randomForest package, which builds binary classification decision trees until enhancers are effectively separated from non-enhancers. Validate on the held-out test classes and compute accuracy with a confusion matrix from the caret package; the acceptance criterion is enhancer prediction-model accuracy slightly above 93%.

## Related tools

- **CIPHER** (Workflow platform exposing the enhancer-prediction task via its analysis mode ('--function predictEnhancers'), combining ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data) — github.com__c-guzman__cipher-workflow-platform
- **randomForest** (R package used to train the RFT model, generating binary classification decision trees until enhancers are separated from non-enhancers)
- **caret** (R package used to compute the confusion matrix and validate the trained model on held-out test classes)
- **Nextflow** (Workflow language that orchestrates the CIPHER enhancer-prediction run) — https://www.nextflow.io/
- **Docker** (Container-based virtualization technology used by CIPHER to package and reproduce the analysis software)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --config <CONFIG> --function predictEnhancers
```

## Evaluation signals

- Held-out test accuracy from the caret confusion matrix is slightly above 93%, i.e. the majority of 'true' enhancers are identified during the testing stage
- The number of positive and negative training elements is balanced (equal number of ENCODE enhancers and -1/+1 kb promoter regions)
- Features are computed as average coverage signal per marker within a set distance along each genomic element from RPM-normalized bedGraph files
- One third of the classes and their classifications are held out for testing, and the saved model can be reloaded and re-scored to reproduce the same confusion matrix

## Limitations

- Accuracy is defined relative to ENCODE enhancers as positives and promoters (-1/+1 kb from TSS) as negatives, so performance on other element classes is not established
- Feature values depend on RPM-normalized bedGraph signal files and a fixed distance window along each element; changes to normalization or window distance alter model behavior
- The model currently only runs as part of entire CIPHER workflows — CIPHER does not yet allow running only a subset of tools
- No modular tool selection or multi-design/time-series RNA-seq support exists in CIPHER, so integration steps around enhancer prediction are constrained by the workflow platform

## Evidence

- [methods] RFT model training and accuracy: "CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%"
- [methods] predictEnhancers function: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
- [methods] Positive class definition: "one class containing a previously determined set of enhancer elements from the Encyclopedia of DNA Elements (ENCODE) project"
- [readme] CIPHER platform scope: "CIPHER is a data processing workflow platform for next generation sequencing data including ChIP-seq, RNA-seq, DNase-seq, MNase-seq, ATAC-seq and GRO-seq."
- [abstract] Analysis mode for enhancer discovery: "CIPHER contains an "analysis" mode that completes complex bioinformatics tasks such as enhancer discovery and provides functions to integrate various datasets together."
