---
name: binomial-significance-testing
description: Use when reproducing or extending DESSO's TFBS identification step, where
  motif instance candidates from a CNN motif detector must be statistically assessed
  with binomial P-values at varying signal cutoffs instead of a fixed subjective cutoff.
when_to_use_negative:
- The analysis goal is training the CNN or predicting motif patterns themselves —
  this skill only covers post-prediction statistical selection of motif instances
  (TFBSs), not model training or motif discovery via predict.py.
- Motif detectors come from tools other than DESSO (e.g. DeepBind, Basset, MEME-ChIP)
  whose outputs are not motif-instance candidate sets at a cutoff grid compatible
  with Omega(M, lambda) construction.
- The question concerns DNA shape integration or GCNN binding specificity prediction,
  which are separate DESSO workflows in the README.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DESSO
  role: Provides the CNN motif detectors and predict.py motif-instance outputs (cutoff
    interval 0.01–1, step 0.03) that the binomial model evaluates; the binomial distribution
    is integrated into DESSO to optimize TFBS identification.
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:05b114d8ea2883b7aaf9eaf4387dc8373d77fc06761e008df1f3fe209b919910
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OSU-BMBL/DESSO
  repo_url: https://github.com/OSU-BMBL/DESSO
  tools:
  - DESSO
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# binomial-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct DESSO's binomial-distribution statistical model, which replaces a subjective motif signal cutoff with the selection of TFBS candidate sets whose binomial P-value is minimal, enabling objective motif instance identification from CNN motif detectors.

## When to use

Use when reproducing or extending DESSO's TFBS identification step, where motif instance candidates from a CNN motif detector must be statistically assessed with binomial P-values at varying signal cutoffs instead of a fixed subjective cutoff.

## When NOT to use

- The analysis goal is training the CNN or predicting motif patterns themselves — this skill only covers post-prediction statistical selection of motif instances (TFBSs), not model training or motif discovery via predict.py.
- Motif detectors come from tools other than DESSO (e.g. DeepBind, Basset, MEME-ChIP) whose outputs are not motif-instance candidate sets at a cutoff grid compatible with Omega(M, lambda) construction.
- The question concerns DNA shape integration or GCNN binding specificity prediction, which are separate DESSO workflows in the README.

## Inputs

- GC-matched background set of 500,000 101-bp bins (encode_101_background, unpacked into data/)
- DESSO predict.py motif-instance outputs generated with --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
- Per-motif occurrence probabilities estimated from the background bins

## Outputs

- Per-motif, per-cutoff binomial P-value table (binomial_pvalues.tsv) with pass/fail retention decisions per motif
- Optimal motif instance set Omega(M, lambda_min) per motif (minimum-P-value candidates)
- P-value validation report (pvalue_validation_report.tsv) recording deviations from an independent binomial-CDF reference

## How to apply

Load the GC-matched background set of 500,000 101-bp bins (encode_101_background) into data/ and parse the DESSO predict.py motif-instance outputs generated with start_cutoff 0.01, end_cutoff 1, and step_cutoff 0.03. For each motif M and cutoff lambda, build the instance set Omega(M, lambda) of peak sequences whose predicted binding sites pass that cutoff, and estimate per-motif occurrence probabilities from the 500,000 background bins. Compute binomial P-values (formula 5 of the paper) for each Omega(M, lambda) using an exact binomial CDF implementation, then select the candidate set corresponding to the minimum P-value as the optimal motif instances. Apply the paper's retention filters — motifs with P < 1e-4, more than 5 motif instances, and at least 3 positions with information content > 1 — and validate computed P-values against an independent binomial-CDF reference implementation at fixed lambda values.

## Related tools

- **DESSO** (Provides the CNN motif detectors and predict.py motif-instance outputs (cutoff interval 0.01–1, step 0.03) that the binomial model evaluates; the binomial distribution is integrated into DESSO to optimize TFBS identification.) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The per-motif minimum P-value candidate set corresponds to a cutoff lambda within the swept interval [0.01, 1] at step 0.03 from predict.py.
- All retained motifs satisfy the three paper filters: P < 1e-4, > 5 motif instances, and ≥ 3 positions with information content > 1.
- Computed binomial P-values agree with an independent binomial-CDF reference implementation at fixed lambda values, with deviations recorded (and near-zero) in pvalue_validation_report.tsv.
- Per-motif occurrence probabilities are consistent with frequencies measured over the full 500,000 background 101-bp bins.
- binomial_pvalues.tsv contains one row per motif and per cutoff with both the P-value and the pass/fail retention decision.

## Limitations

- P-value computation depends on the GC-matched background bin set; changes to the background (e.g. non-human genomes, different bin lengths than 101 bp) invalidate the estimated occurrence probabilities.
- The paper notes existing DL-based methods, including the motif detectors feeding this model, assume each motif position contributes independently, which may underestimate nucleotide dependencies.
- The README ships only wgEncodeEH002288 example data due to file size limits; the full datasets (~5.9GB) must be downloaded separately, and Python 2.7 / TensorFlow 1.1.0 / CUDA 8.0.44 are required.

## Evidence

- [intro] we integrated the binomial distribution into DESSO to optimize the TFBSs identification based on identified motif patterns: "we integrated the binomial distribution into DESSO to optimize the TFBSs identification based on identified motif patterns"
- [readme] python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03: "python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03"
- [readme] Download [encode_101_background](https://bmblx.bmi.osumc.edu/downloadFiles/DESSO/encode_101_background.zip), then unzip them and put them into `data/`: "and [encode_101_background](https://bmblx.bmi.osumc.edu/downloadFiles/DESSO/encode_101_background.zip), then unzip them and put them into `data/`"
- [methods] DESSO's binomial-based statistical model determines the optimal motif instances for each motif detector by computing a binomial P-value for motif instance candidate sets at varying signal cutoffs: [evidence span withheld — anchor status: not_found]
- [abstract] we developed a new method (DEep Sequence and Shape mOtif or DESSO) for cis-regulatory motif prediction using deep neural networks and the binomial distribution model: "we developed a new method (DEep Sequence and Shape mOtif or DESSO) for cis-regulatory motif prediction using deep neural networks and the binomial distribution model"
