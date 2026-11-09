---
name: encode-enhancer-region-annotation
description: Use when ChIP-seq (H3K4me1, H3K4me3, H3K27ac) and DNase-seq RPM-normalized
  bedGraph signals are available and genomic regions must be classified as enhancers
  versus promoters, e.g. enhancer discovery in a human cell line.
when_to_use_negative:
- Input is already a curated enhancer list or feature table rather than raw/processed
  NGS signal from which bedGraph coverage features must be computed
- The analysis goal is RNA-seq differential expression or read-mapping QC rather than
  genomic regulatory element classification
- Only DNase-seq data is available without the required H3K4me1/H3K4me3/H3K27ac ChIP-seq
  signatures
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: CIPHER
  role: Workflow platform providing the 'predictEnhancers' analysis function that
    combines ChIP-seq and DNase-seq signals to predict enhancers
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: randomForest
  role: R package used to train the binary classification decision trees separating
    enhancers from non-enhancers
- name: caret
  role: R package used to compute the confusion matrix and accuracy on the held-out
    test set
- name: Nextflow
  role: Workflow language in which the CIPHER enhancer-prediction pipeline is implemented
    and executed
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:cdb35685f3ca6e99531ad0f9710a7fa4dcad5ac93d18f05f7045c6decd4f1e8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/c-guzman/cipher-workflow-platform
  tools:
  - CIPHER
  - Docker
  - Nextflow
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# encode-enhancer-region-annotation

## Summary

Trains and validates CIPHER's random-forest-based enhancer prediction model, which classifies genomic elements as enhancers versus promoters using chromatin signatures (H3K4me1, H3K4me3, H3K27ac) and DNase-seq DNase hypersensitive site signal, achieving accuracy slightly above 93%.

## When to use

Use when ChIP-seq (H3K4me1, H3K4me3, H3K27ac) and DNase-seq RPM-normalized bedGraph signals are available and genomic regions must be classified as enhancers versus promoters, e.g. enhancer discovery in a human cell line.

## When NOT to use

- Input is already a curated enhancer list or feature table rather than raw/processed NGS signal from which bedGraph coverage features must be computed
- The analysis goal is RNA-seq differential expression or read-mapping QC rather than genomic regulatory element classification
- Only DNase-seq data is available without the required H3K4me1/H3K4me3/H3K27ac ChIP-seq signatures

## Inputs

- Previously determined enhancer elements from the ENCODE project (positives)
- Promoter regions spanning -1/+1 kb from the TSS of known genes (negatives)
- RPM-normalized bedGraph signal files for DNase-seq (DHS), H3K4me1, H3K4me3, and H3K27ac ChIP-seq
- GTF/gene annotation for defining promoter TSS regions

## Outputs

- Trained random-forest enhancer-classification model (saved RFT model)
- Confusion-matrix-derived prediction accuracy metric (expected slightly above 93%)
- Set of predicted potential enhancers

## How to apply

Load two genomic element classes: previously determined ENCODE enhancer elements as positives and an equal number of promoter regions spanning -1/+1 kb from the transcription start site of known genes as negatives. Compute model features as the average coverage signal of each marker (DNase-seq DHS plus H3K4me1, H3K4me3, and H3K27ac) within a set distance along each genomic element, using RPM-normalized bedGraph signal files. Hold out one third of the enhancer and non-enhancer elements with their classifications for testing, and train the RFT model in R (version 3.3.1) with the randomForest package, generating binary classification decision trees until the model separates enhancers from non-enhancers. Validate on the held-out test set and compute accuracy with a confusion matrix from the caret package in R. CIPHER exposes this via the '--function predictEnhancers' analysis-mode step, which combines ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data.

## Related tools

- **CIPHER** (Workflow platform providing the 'predictEnhancers' analysis function that combines ChIP-seq and DNase-seq signals to predict enhancers) — https://github.com/c-guzman/cipher-workflow-platform
- **randomForest** (R package used to train the binary classification decision trees separating enhancers from non-enhancers)
- **caret** (R package used to compute the confusion matrix and accuracy on the held-out test set)
- **Nextflow** (Workflow language in which the CIPHER enhancer-prediction pipeline is implemented and executed) — https://www.nextflow.io/

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --config CONFIG --function predictEnhancers
```

## Evaluation signals

- Held-out test accuracy from the confusion matrix should be slightly above 93%, i.e. the majority of 'true' enhancers are identified during the testing stage
- Training and testing sets must be disjoint: one third of the enhancer and non-enhancer elements (with their classifications) are held out for testing
- The trained randomForest model object must be saved and the accuracy written to a metrics record
- Feature computation must use RPM-normalized bedGraph signals averaged within a set distance along each genomic element, not raw read counts

## Limitations

- Validation was demonstrated in human cell lines using ENCODE-derived enhancers; model performance on other organisms or poorly characterized cell types is not established
- Feature signal depends on a 'set distance' along each genomic element, so results are sensitive to that windowing choice
- The repository README indicates CIPHER only runs entire workflows, not subsets of tools, so enhancer prediction cannot be run in isolation without the surrounding pipeline steps

## Evidence

- [methods] CIPHER's enhancer-prediction model is a random-forest based classifier that predicts transcriptional enhancers using a combination of chromatin signatures: [evidence span withheld — anchor status: not_found]
- [methods] predictEnhancers: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
- [methods] ENCODE enhancers and promoter negatives: "one class containing a previously determined set of enhancer elements from the Encyclopedia of DNA Elements (ENCODE) project [40]"
- [abstract] analysis mode for enhancer discovery: "CIPHER contains an "analysis" mode that completes complex bioinformatics tasks such as enhancer discovery and provides functions to integrate various datasets together."
- [intro] validation goal of the study: "we demonstrate that CIPHER is a fast, reproducible, and flexible tool that accurately processes and integrates NGS datasets"
