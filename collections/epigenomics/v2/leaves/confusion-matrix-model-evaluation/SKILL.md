---
name: confusion-matrix-model-evaluation
description: Use when a trained binary classifier (e.g. randomForest RFT for enhancer
  vs. non-enhancer elements) must be validated on held-out test classes and its accuracy
  quantified via a confusion matrix.
when_to_use_negative:
- You only have a single pooled dataset with no held-out class-balanced test split
  — accuracy on training data would be inflated and the 93% criterion does not apply.
- The classification task is multi-class or the goal is class-specific metrics (precision/recall
  per class) beyond the binary enhancer/non-enhancer accuracy reported here.
- Input is raw NGS signal (bedGraph/RPM tracks) that has not yet been converted into
  per-element average coverage features for the classifier.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3169
tools:
- name: randomForest
  role: R package used to train the RFT binary classification decision trees that
    the confusion matrix evaluates
  repo: c-guzman/cipher-workflow-platform
- name: caret
  role: R package providing the confusionMatrix function from which testing accuracy
    is computed
- name: CIPHER
  role: Workflow platform whose predictEnhancers function trains and validates the
    random-forest enhancer model
  repo: c-guzman/cipher-workflow-platform
- name: Nextflow
  role: Workflow language in which the CIPHER enhancer-prediction pipeline is executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-017-1770-1
    title: 'CIPHER: a flexible and extensive workflow platform for integrative next-generation
      sequencing data analysis and genomic regulatory element prediction'
  source_content_hash: sha256:c90e5fa36cc12967be268202e40911dd319de263d4144a1268c5f8c4e6b558c0
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# confusion-matrix-model-evaluation

## Summary

Evaluates a trained binary classifier (e.g. CIPHER's random-forest enhancer predictor) by scoring it on a held-out test set and computing accuracy from a confusion matrix, accepting performance slightly above 93%.

## When to use

Use when a trained binary classifier (e.g. randomForest RFT for enhancer vs. non-enhancer elements) must be validated on held-out test classes and its accuracy quantified via a confusion matrix.

## When NOT to use

- You only have a single pooled dataset with no held-out class-balanced test split — accuracy on training data would be inflated and the 93% criterion does not apply.
- The classification task is multi-class or the goal is class-specific metrics (precision/recall per class) beyond the binary enhancer/non-enhancer accuracy reported here.
- Input is raw NGS signal (bedGraph/RPM tracks) that has not yet been converted into per-element average coverage features for the classifier.

## Inputs

- Trained randomForest (RFT) model object (R, randomForest package)
- Held-out test subset (one third) of enhancer positives (ENCODE enhancer elements) and promoter negatives (-1/+1 kb from TSS)
- Feature matrix of average RPM-normalized coverage signal per element for DNase-seq DHS, H3K4me1, H3K4me3, and H3K27ac

## Outputs

- Confusion matrix of true vs. predicted classes (caret package)
- Model accuracy metric written to a metrics record
- Saved trained random-forest model file

## How to apply

Split the labeled elements so that one third of each class (e.g. ENCODE enhancers as positives and -1/+1 kb promoter regions as negatives) is held out for testing while the remainder trains the model. Train the random-forest model in R (version 3.3.1) with the randomForest package, which generates binary classification decision trees until enhancers are effectively separated from non-enhancers. Apply the fitted model to the held-out test classes to obtain predicted labels. Build a confusion matrix of true versus predicted classes using the caret package in R and derive overall accuracy from it. Judge success by the acceptance criterion used in CIPHER: enhancer prediction-model accuracy slightly above 93%, i.e. the majority of 'true' enhancers are identified during the testing stage.

## Related tools

- **randomForest** (R package used to train the RFT binary classification decision trees that the confusion matrix evaluates) — c-guzman/cipher-workflow-platform
- **caret** (R package providing the confusionMatrix function from which testing accuracy is computed)
- **CIPHER** (Workflow platform whose predictEnhancers function trains and validates the random-forest enhancer model) — c-guzman/cipher-workflow-platform
- **Nextflow** (Workflow language in which the CIPHER enhancer-prediction pipeline is executed)

## Examples

```
nextflow run cipher.nf -with-singularity cipher.img --mode analysis --config CONFIG --function predictEnhancers
```

## Evaluation signals

- Confusion-matrix-derived accuracy on the held-out test classes is slightly above 93%.
- The majority of 'true' enhancers in the test set are correctly identified (true-positive quadrant dominates).
- Test split is exactly one third of each class, with classes balanced between enhancers and promoter negatives.
- A saved model file and a metrics record containing the accuracy are produced by the workflow.
- Features were computed as average RPM-normalized bedGraph coverage per marker (DHS, H3K4me1, H3K4me3, H3K27ac) within a set distance of each element.

## Limitations

- Model is validated only on the binary enhancer-vs-promoter task with ENCODE enhancer elements as positives; performance on other element classes is untested.
- The README notes CIPHER runs entire workflows rather than subsets of tools, so the evaluation step cannot be run standalone within CIPHER.
- The pipeline mixes in-memory constraints: running outside a cluster requires limiting parallel processes with the -qs flag or the workflow can abruptly end by running out of memory.
- Feature signal depends on RPM-normalized bedGraph inputs and a fixed distance window; changes to normalization or window size are not covered by the reported 93% accuracy.

## Evidence

- [methods] Train the RFT model in R (version 3.3.1) with the randomForest package, generating binary classification decision trees: [evidence span withheld — anchor status: not_found]
- [methods] Validate the generated RFT model on the held-out test classes and compute accuracy using a confusion matrix from the caret package in R.: [evidence span withheld — anchor status: not_found]
- [methods] CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%: "CIPHER’s enhancer prediction-model accuracy achieved slightly above 93%"
- [methods] holding out one third of the classes and their classifications for testing: [evidence span withheld — anchor status: not_found]
- [methods] Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data: "Enhancer prediction ("--function predictEnhancers") - predicts potential enhancers using a combination of ChIP-seq (H3K4me3, H3K4me1, H3K27Ac) and DNase-seq data"
