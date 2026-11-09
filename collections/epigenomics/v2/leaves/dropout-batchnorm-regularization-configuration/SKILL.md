---
name: dropout-batchnorm-regularization-configuration
description: Use when constructing or re-training the EpiCas-DL CNN (150 multi-size
  convolutional filters, MaxPooling, three 80/60/40 dense layers) and you need its
  batch-normalization + dropout(0.4) regularization block.
when_to_use_negative:
- Building a different architecture (e.g. the README's sequence-only or epigenetic
  prediction models) without the 150-filter convolutional layer — the 0.4 dropout
  rate and batchnorm placement are specific to the EpiCas-DL CNN described here
- Inputs are already normalized activity-score tables for non-deep-learning models
  (e.g. Gradient Boosting or Random Forest baselines in the comparison workflow),
  where batchnorm/dropout do not apply
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: EpiCas-DL
  role: Source architecture whose CNN uses batch normalization and dropout rate 0.4
    after MaxPooling to prevent overfitting
  repo: https://github.com/yangqianq/EpiCas-DL
- name: TensorFlow 2.4.1
  role: Backend used to implement and compile the CNN with batchnorm + dropout(0.4)
    and save it as .h5
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.034
    title: '<i>EpiCas-DL</i> : Predicting sgRNA activity for CRISPR-mediated epigenome
      editing by deep learning'
  source_content_hash: sha256:118eb38c9c85bfb0291cd2ec4882be852a8ff60c0dc129eddcee9930aecfa650
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/yangqianq/EpiCas-DL
  tools:
  - EpiCas-DL
  - Python 3.6
  - TensorFlow 2.4.1
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# dropout-batchnorm-regularization-configuration

## Summary

Configure regularization for the EpiCas-DL CNN by inserting batch normalization followed by a dropout layer with rate 0.4 immediately after MaxPooling, to curb overfitting when training on epigenetic sgRNA screening data.

## When to use

Use when constructing or re-training the EpiCas-DL CNN (150 multi-size convolutional filters, MaxPooling, three 80/60/40 dense layers) and you need its batch-normalization + dropout(0.4) regularization block.

## When NOT to use

- Building a different architecture (e.g. the README's sequence-only or epigenetic prediction models) without the 150-filter convolutional layer — the 0.4 dropout rate and batchnorm placement are specific to the EpiCas-DL CNN described here
- Inputs are already normalized activity-score tables for non-deep-learning models (e.g. Gradient Boosting or Random Forest baselines in the comparison workflow), where batchnorm/dropout do not apply

## Inputs

- 11 x 40 encoded sgRNA feature matrix (4 sequence channels, 4 nucleosome positioning channels, 1 chromatin accessibility, 1 DNA methylation, 1 gene expression)
- CNN architecture specification (150 multi-size filters, MaxPooling, 80/60/40 dense layers)
- Python 3.6/3.7 environment with TensorFlow 2.4.1

## Outputs

- Compiled Keras/TF model with batch normalization and dropout(rate=0.4) after MaxPooling, saved as epicas_dl_cnn_model.h5
- model.summary() confirming the regularization layers and dense layer sizes

## How to apply

After defining the 11x40 encoded sgRNA input tensor and the single convolutional layer with 150 filters (30 each of 1-5 nt sizes, ReLU), apply MaxPooling, then add a batch normalization layer, and follow it with a dropout layer at rate 0.4 — this ordering (MaxPooling → batchnorm → dropout 0.4) is how EpiCas-DL prevents overfitting. Continue with three fully connected layers of 80, 60, and 40 units (ReLU) and a linear-activation output layer. Build and compile the model in TensorFlow 2.4.1 (the backend named in the article), then save the compiled architecture to an .h5 file such as epicas_dl_cnn_model.h5. Validation: load the .h5 model and confirm its summary shows the batch-normalization and dropout(0.4) layers positioned after MaxPooling and before the 80/60/40 dense stack.

## Related tools

- **EpiCas-DL** (Source architecture whose CNN uses batch normalization and dropout rate 0.4 after MaxPooling to prevent overfitting) — https://github.com/yangqianq/EpiCas-DL
- **TensorFlow 2.4.1** (Backend used to implement and compile the CNN with batchnorm + dropout(0.4) and save it as .h5)

## Examples

```
python example.py -sequence GGGCCCAGATCCCTCTATGTGCTCGAAGCAGGTGGACCCC -model CRISPRoff_epi -chrom chr12 -start 100592057 -end 100592096 -pamCoord 100592087 -transcript P1P2 -gene ACTR6 -cell_line Hek293t
```

## Evaluation signals

- model.summary() lists a BatchNormalization layer and a Dropout layer with rate 0.4 located after MaxPooling and before the 80/60/40 dense stack
- The saved epicas_dl_cnn_model.h5 file exists and loads successfully with TensorFlow 2.4.1
- Model compiles without error under TensorFlow 2.4.1 (article: Tensorflow 2.4.1 used as backend for deep learning models)
- Convolutional layer still contains 150 multi-size filters (30 each of 1-5 nt), confirming the regularization block did not alter feature extraction

## Limitations

- Small CRISPRoff screening datasets mainly in HEK293T and K562 cell lines increase overfitting risk and limit generalization — the dropout 0.4 setting mitigates but does not remove this
- Only nucleosome positioning, chromatin accessibility, DNA methylation, and RNA expression are encoded; incomplete epigenetic feature information limits model performance
- Article notes other more complex deep learning frameworks (RNNs, GANs, RL) remain unexplored; this regularization recipe is specific to the EpiCas-DL CNN

## Evidence

- [full_text] MaxPooling, then batch normalization followed by dropout with rate 0.4 to prevent overfitting: [evidence span withheld — anchor status: not_found]
- [methods] Tensorflow 2.4.1 backend for deep learning models: "Scikit-learn 0.24.2 and Tensorflow 2.4.1 were used as the backend for machine learning algorithms and deep learning models"
- [discussion] Small CRISPRoff screening datasets may increase overfitting risk: "The amount of available CRISPRoff screening datasets were relatively small and mainly conducted in HEK293T and K562 cell lines, which may increase the overfitting risk"
