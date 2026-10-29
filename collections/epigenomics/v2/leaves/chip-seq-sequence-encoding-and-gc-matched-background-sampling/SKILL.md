---
name: chip-seq-sequence-encoding-and-gc-matched-background-sampling
description: Use when predicting TF-DNA binding specificity from ENCODE ChIP-seq peaks
  with long DNA sequences (1001 bp) and a gated CNN, or when comparing GCNN AUCs against
  the 101-bp one-layer CNN baseline.
when_to_use_negative:
- Goal is motif prediction with the one-layer CNN on 101-bp peaks (use --peak_flank
  50 --network CNN instead of GCNN with peak_flank 500).
- 'Input is DNA shape features (DNAShape: HelT, MGW, ProT, Roll) rather than DNA sequences
  — use --feature_format DNAShape, not Seq.'
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Deep learning framework whose train.py trains the GCNN on GC-matched 1001-bp
    sequence windows and reports test AUC
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:f3518556f07c56be4b52662e2c87a44bf0a9be74fa8bc83f71aa953ac6b7b4f2
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
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# chip-seq-sequence-encoding-and-gc-matched-background-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assemble GC-matched long-sequence ChIP-seq inputs (e.g., 1001-bp peak windows from data/encode_1001 plus GRCh37.p13.genome.fa) for training DESSO's gated-CNN (GCNN) to predict TF–DNA binding specificity, evaluating with test-set ROC AUC.

## When to use

Use when predicting TF-DNA binding specificity from ENCODE ChIP-seq peaks with long DNA sequences (1001 bp) and a gated CNN, or when comparing GCNN AUCs against the 101-bp one-layer CNN baseline.

## When NOT to use

- Goal is motif prediction with the one-layer CNN on 101-bp peaks (use --peak_flank 50 --network CNN instead of GCNN with peak_flank 500).
- Input is DNA shape features (DNAShape: HelT, MGW, ProT, Roll) rather than DNA sequences — use --feature_format DNAShape, not Seq.

## Inputs

- GC-matched long-sequence peak dataset in data/encode_1001
- GRCh37.p13.genome.fa reference genome in data/
- ENCODE ChIP-seq dataset indices (690 datasets, e.g., wgEncodeEH002288)

## Outputs

- Trained GCNN model at output/encode_1001/gc_match/<dataset>/Seq/GCNN
- Test_result.txt with area under the ROC curve (AUC) on the test split

## How to apply

Place the GC-matched long-sequence input from data/encode_1001 and the GRCh37.p13.genome.fa reference into data/. Configure the run indices over the 690 ENCODE ChIP-seq datasets via --start_index/--end_index (e.g., --start_index 0 --end_index 1 for wgEncodeEH002288), and train the gated-CNN by invoking train.py with --network GCNN --peak_flank 500 --feature_format Seq, which yields 1001-bp peak windows (2*500+1). The trained model and its AUC are written to output/encode_1001/gc_match/<dataset>/Seq/GCNN with Test_result.txt. Repeat across the dataset index range and compare per-dataset GCNN AUCs against the one-layer CNN baseline trained on 101-bp sequences (--peak_flank 50) to quantify the benefit of gating and long-range context.

## Related tools

- **DESSO** (Deep learning framework whose train.py trains the GCNN on GC-matched 1001-bp sequence windows and reports test AUC) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 500 --network GCNN --feature_format Seq
```

## Evaluation signals

- Test_result.txt is written in output/encode_1001/gc_match/<dataset>/Seq/GCNN and contains a valid ROC AUC value.
- The peak length is exactly (2 * 500 + 1) = 1001 base pairs when --peak_flank 500 is used.
- Per-dataset GCNN AUCs aggregated across the --start_index/--end_index range can be compared against the 101-bp CNN baseline AUCs.
- The correct dataset index range maps to the 690 ENCODE ChIP-seq datasets (index 0 = wgEncodeEH002288).

## Limitations

- The repo ships only wgEncodeEH002288-related data for encode_101/encode_1001/TfbsUniform_hg19_ENCODE due to file size limits; the whole ~5.9GB dataset must be downloaded separately.
- Sequence showed little or negative effect from multi-convolutional layers — architectural complexity helps DNA shape features, not necessarily sequence.
- DEPSO-era dependencies (Tensorflow 1.1.0, CUDA 8.0.44, Python 2.7) are dated and may be hard to install.

## Evidence
