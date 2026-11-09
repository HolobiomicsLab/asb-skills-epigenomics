---
name: ht-selex-enrichment-data-preparation
description: Use when input HT-SELEX data must be converted into per-cycle enrichment
  files (proteins/{protein}/{cycle}.txt) for regressor training, and inconsistent
  enrichment cases across cycles need to be filtered out before feature extraction.
when_to_use_negative:
- Input data is uPBM or gcPBM rather than HT-SELEX — uPBM requires MEME-suite-based
  sequence trimming and upbm_regressor.py, and gcPBM requires {protein}_{concentration}.txt
  plus _freq_matrix_6.txt with gcpbm_regressor.py
- Only a single HT-SELEX cycle is available, since the SVM filter relies on comparing
  enrichments across different cycles to flag inconsistencies
- Sequences are already a curated training matrix from another pipeline with enrichment
  values precomputed and no cross-cycle inconsistency risk
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: DNAffinity
  role: Pipeline whose preprocessing notebooks and selex_regressor.py consume the
    prepared per-cycle HT-SELEX enrichment files
  repo: https://github.com/Jalbiti/DNAffinity
- name: MEME suite
  role: Used in the DNAffinity preprocessing workflow for sequence trimming (for uPBM
    data in the same pipeline; HT-SELEX preparation itself relies on the SVM cross-cycle
    filter)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkac708
    title: 'DNAffinity: a machine-learning approach to predict DNA binding affinities
      of transcription factors'
  source_content_hash: sha256:16c597862af616e888383f26cb915988b44bdc98dfc8fa8b87e7860ca4ed715c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Jalbiti/DNAffinity
  repo_url: https://github.com/Jalbiti/DNAffinity
  tools:
  - BEESEM
  - BindSpace
  - DNAffinity
  - DeepBind
  - MEME suite
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ht-selex-enrichment-data-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare HT-SELEX per-cycle enrichment data for DNAffinity random-forest regressor training: verify per-cycle input files, apply SVM filtering to discard cases with inconsistent enrichments across HT-SELEX cycles, and format files so selex_regressor.py can extract DNA physical-property features. This step prevents suspicious cross-cycle inconsistencies from biasing model training.

## When to use

Use when input HT-SELEX data must be converted into per-cycle enrichment files (proteins/{protein}/{cycle}.txt) for regressor training, and inconsistent enrichment cases across cycles need to be filtered out before feature extraction.

## When NOT to use

- Input data is uPBM or gcPBM rather than HT-SELEX — uPBM requires MEME-suite-based sequence trimming and upbm_regressor.py, and gcPBM requires {protein}_{concentration}.txt plus _freq_matrix_6.txt with gcpbm_regressor.py
- Only a single HT-SELEX cycle is available, since the SVM filter relies on comparing enrichments across different cycles to flag inconsistencies
- Sequences are already a curated training matrix from another pipeline with enrichment values precomputed and no cross-cycle inconsistency risk

## Inputs

- HT-SELEX per-cycle enrichment files at proteins/{protein}/{cycle}.txt
- Protein (transcription factor) name passed as 1st CLI argument
- HT-SELEX cycle number passed as 2nd CLI argument
- Template-formatted reference files from the DNAffinity test_data folder (e.g. Gata4)

## Outputs

- Consistency-filtered, template-conformant per-cycle HT-SELEX files ready for selex_regressor.py feature extraction
- Trained random-forest regressor on HT-SELEX affinities (downstream of this preparation step)
- Preprocessing notebooks/logs documenting per-cycle enrichment handling

## How to apply

Collect the HT-SELEX per-cycle enrichment files and verify their formatting against the repository templates, placing them at proteins/{protein}/{cycle}.txt. Apply an SVM-based filter to remove cases displaying inconsistency between the enrichments at different HT-SELEX cycles, since these are suspicious and would bias training and testing of the random forest regressor on DNA physical properties (equilibrium geometry and flexibility at base-pair step resolution derived from MD simulations). Decide the cycle number to train on and pass it as the second argument when launching selex_regressor.py, with the protein name as the first argument. The prepared per-cycle files then feed feature extraction and affinity training; sanity-check the pipeline using the test_data folder's Gata4 example.

## Related tools

- **DNAffinity** (Pipeline whose preprocessing notebooks and selex_regressor.py consume the prepared per-cycle HT-SELEX enrichment files) — https://github.com/Jalbiti/DNAffinity
- **MEME suite** (Used in the DNAffinity preprocessing workflow for sequence trimming (for uPBM data in the same pipeline; HT-SELEX preparation itself relies on the SVM cross-cycle filter))

## Examples

```
python selex_regressor.py Gata4 4
```

## Evaluation signals

- Every protein directory contains per-cycle files named proteins/{protein}/{cycle}.txt whose formatting matches the repository templates
- SVM filter flags and removes cases whose enrichment values are inconsistent across HT-SELEX cycles before training
- selex_regressor.py runs without errors when given the protein name (1st argument) and cycle number (2nd argument), e.g. 'python selex_regressor.py Gata4 4'
- Downstream cross-dataset benchmark reproduces DNAffinity's average Pearson correlation of ~0.47 on HT-SELEX-to-uPBM transfer, exceeding DeepBind (0.35), BEESEM (0.36) and BindSpace (0.20)
- Test-case execution on the bundled Gata4 example completes before applying the workflow to new proteins

## Limitations

- The SVM filter depends on enrichment values from multiple HT-SELEX cycles; single-cycle or incompletely cycled data cannot be checked for cross-cycle consistency
- Cases with inconsistent enrichments across cycles are treated as suspicious and discarded, so genuine biological signal in those cases is lost
- The transcription factor binding site concept is not uniquely defined and depends on the intrinsically noisy, low-resolution experimental technique used, so prepared HT-SELEX affinities may not transfer universally
- Training/test splits used 80% of data for training and 20% for evaluation, so results are sensitive to this partition

## Evidence
