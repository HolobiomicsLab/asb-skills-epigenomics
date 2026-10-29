---
name: genomic-feature-engineering-from-chromatin-signals
description: Use when genomic elements (candidate enhancers vs promoters) must be
  converted into numeric features from ChIP-seq/DNase-seq bedGraph signals to train
  or apply an enhancer-prediction classifier.
when_to_use_negative:
- Input is raw FASTQ sequence rather than processed RPM-normalized signal — run CIPHER's
  pre-processing pipelines first (trimming, mapping, track generation)
- No H3K4me1/H3K4me3/H3K27ac or DNase-seq data are available for the cell line of
  interest, since features depend on this combination
- A training-free enhancer discovery method is needed; this skill requires labeled
  enhancer and promoter classes
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: CIPHER
  role: Workflow platform invoking the predictEnhancers function that combines ChIP-seq
    and DNase-seq signals for enhancer prediction
  repo: https://github.com/c-guzman/cipher-workflow-platform
- name: randomForest
  role: R package used to train the binary classification decision trees that separate
    enhancers from non-enhancers
- name: caret
  role: R package used to compute the confusion matrix and derived accuracy on held-out
    test classes
- name: Nextflow
  role: Workflow language in which the CIPHER enhancer-prediction step is orchestrated
    and executed
  repo: https://www.nextflow.io/
- name: Docker
  role: Container technology CIPHER uses to package software for reproducible execution
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:148aecf432b6655d6953a0a10e9d0c54eea3a19298e2ef8ed98cff017e1ea790
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# genomic-feature-engineering-from-chromatin-signals

## Summary

Builds per-element features from RPM-normalized chromatin signal coverage (DNase-seq DHS plus H3K4me1, H3K4me3, H3K27ac) averaged within a set distance along each genomic element, to train CIPHER's random-forest enhancer classifier.

## When to use

Use when genomic elements (candidate enhancers vs promoters) must be converted into numeric features from ChIP-seq/DNase-seq bedGraph signals to train or apply an enhancer-prediction classifier.

## When NOT to use

- Input is raw FASTQ sequence rather than processed RPM-normalized signal — run CIPHER's pre-processing pipelines first (trimming, mapping, track generation)
- No H3K4me1/H3K4me3/H3K27ac or DNase-seq data are available for the cell line of interest, since features depend on this combination
- A training-free enhancer discovery method is needed; this skill requires labeled enhancer and promoter classes

## Inputs

- RPM-normalized bedGraph signal files for DNase-seq DHS, H3K4me1, H3K4me3, and H3K27ac
- Previously determined enhancer elements from the ENCODE project (positives)
- Equal number of promoter regions (-1/+1 kb from TSS of known genes) as negatives
- Ensembl/Gencode reference files (downloaded via CIPHER's --download_data flag)

## Outputs

- Feature matrix of average marker coverage per genomic element
- Trained random-forest enhancer-prediction model (saved RFT model)
- Confusion matrix and derived classification accuracy metric record

## How to apply

Load two element classes: ENCODE-derived enhancer elements as positives and an equal number of promoter regions spanning -1/+1 kb around known-gene transcription start sites as negatives. Compute model features as the average coverage signal of each chromatin marker (DNase-seq DHS plus H3K4me1, H3K4me3, and H3K27ac) within a set distance along each genomic element, using RPM-normalized bedGraph signal files. Split elements into training and testing datasets, holding out one third for testing. Train the RFT model in R (version 3.3.1) with the randomForest package, generating binary classification decision trees, then validate on the held-out classes and compute accuracy via a confusion matrix from the caret package.

## Related tools

- **CIPHER** (Workflow platform invoking the predictEnhancers function that combines ChIP-seq and DNase-seq signals for enhancer prediction) — https://github.com/c-guzman/cipher-workflow-platform
- **randomForest** (R package used to train the binary classification decision trees that separate enhancers from non-enhancers)
- **caret** (R package used to compute the confusion matrix and derived accuracy on held-out test classes)
- **Nextflow** (Workflow language in which the CIPHER enhancer-prediction step is orchestrated and executed) — https://www.nextflow.io/
- **Docker** (Container technology CIPHER uses to package software for reproducible execution)

## Examples

```
nextflow run cipher.nf -with-singularity <cipher.img> --mode analysis --function predictEnhancers --config <CONFIG> --fa <FASTA> --gtf <GTF>
```

## Evaluation signals

- Confusion-matrix accuracy of the RFT model on the held-out one-third test set is slightly above 93%
- Majority of 'true' enhancers are correctly identified during the testing stage
- Feature values equal the average RPM-normalized bedGraph coverage signal per marker within the set distance of each element
- Positive and negative classes are balanced (equal numbers of enhancers and promoter regions)
- Trained model and confusion-matrix-derived accuracy are written to a saved metrics record

## Limitations

- Accuracy depends on cell-line-matched ENCODE signal data; the built-in validation used ENCODE enhancers and promoter regions as classes
- The enhancer-prediction model relies specifically on the combination of H3K4me1, H3K4me3, H3K27ac ChIP-seq and DNase-seq; other marks are not integrated
- CIPHER currently runs entire workflows rather than subsets of tools, so re-running only the enhancer-prediction step is not supported
- Memory must be managed when running the workflow locally; too many parallel processes will cause the workflow to end abruptly

## Evidence

- [methods] Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
- [methods] Compute model features as the average coverage signal of each chromatin marker (DNase-seq DHS plus H3K4me1, H3K4me3, and H3K27ac) within a set distance along each genomic element, using RPM-normalized bedGraph signal files.: [evidence span withheld — anchor status: not_found]
- [methods] Train the RFT model in R (version 3.3.1) with the randomForest package, generating binary classification decision trees until the model can effectively separate enhancers from non-enhancers.: [evidence span withheld — anchor status: not_found]
- [methods] CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%: "CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%"
- [methods] one class containing a previously determined set of enhancer elements from the Encyclopedia of DNA Elements (ENCODE) project: "one class containing a previously determined set of enhancer elements from the Encyclopedia of DNA Elements (ENCODE) project"
- [readme] If not running on a cluster please set the `-qs <INT>` flag in order to control the number of processes that CIPHER parallelizes.: [evidence span withheld — anchor status: external_doc]
