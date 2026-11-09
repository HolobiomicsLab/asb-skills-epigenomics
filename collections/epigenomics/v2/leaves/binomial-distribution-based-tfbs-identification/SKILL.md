---
name: binomial-distribution-based-tfbs-identification
description: Use when predicting sequence or DNA shape motifs and TFBSs from trained
  DESSO CNN models on ENCODE ChIP-seq peaks and an objective, statistically grounded
  motif cutoff is needed instead of a manual threshold.
when_to_use_negative:
- You have not yet trained a DESSO CNN model for the dataset — predict.py depends
  on trained models.
- You need de novo motif discovery from raw peaks without a deep learning framework
  (e.g. MEME-ChIP-style EM approaches) rather than DL-based motif/TFBS prediction.
- Your peaks are not human ENCODE ChIP-seq or lack the 101-bp summit-centered format
  expected with --peak_flank 50 (unless you preprocess your own peaks with processing_peaks.py).
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: Framework providing predict.py with binomial-distribution-based cutoff scanning
    for motif and TFBS identification from trained CNN models
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Backend used to train and run the CNN models whose predictions are scanned
    by the cutoff procedure
- name: Biopython
  role: Dependency for sequence/feature handling in the DESSO pipeline
- name: scikit-learn
  role: Dependency used in DESSO model training/prediction pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:91932b236259623dc2ad37862273be89a359101a72bab5669f2c2061e0d730ab
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
  - Biopython
  - DESSO
  - TensorFlow
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# binomial-distribution-based-tfbs-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

DESSO's statistical approach for identifying TFBSs and motifs from trained CNN models by scanning prediction cutoffs and selecting the cutoff that optimizes binomial-distribution-based significance, replacing subjective cutoff choices.

## When to use

Use when predicting sequence or DNA shape motifs and TFBSs from trained DESSO CNN models on ENCODE ChIP-seq peaks and an objective, statistically grounded motif cutoff is needed instead of a manual threshold.

## When NOT to use

- You have not yet trained a DESSO CNN model for the dataset — predict.py depends on trained models.
- You need de novo motif discovery from raw peaks without a deep learning framework (e.g. MEME-ChIP-style EM approaches) rather than DL-based motif/TFBS prediction.
- Your peaks are not human ENCODE ChIP-seq or lack the 101-bp summit-centered format expected with --peak_flank 50 (unless you preprocess your own peaks with processing_peaks.py).

## Inputs

- Trained DESSO CNN model (output/encode_101/gc_match/<dataset>/Seq/CNN or .../DNAShape/CNN)
- ENCODE ChIP-seq peak data for 690 human datasets (e.g. wgEncodeEH002288) with peak_flank 50 (101-bp peak regions)
- DNA shape feature matrices (HelT, MGW, ProT, Roll) for the peak regions when --feature_format DNAShape
- GRCh37.p13.genome.fa and encode_101_background placed in data/

## Outputs

- Predicted motifs and TFBSs per dataset under output/encode_101/gc_match/<dataset>/<format>/CNN/<index>
- Four shape motif sets: CNN/0 = HelT, CNN/1 = MGW, CNN/2 = ProT, CNN/3 = Roll
- Summary counts (e.g. 1257 HelT, 84 MGW, 885 ProT, 478 Roll motifs; 598 of 690 datasets with ≥1 shape motif)

## How to apply

After training a CNN per dataset (e.g. with train.py --peak_flank 50 --network CNN --feature_format Seq or DNAShape), run DESSO's predict.py with a cutoff scan: --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03. At each candidate cutoff, motif-matching sites are counted and binomial-distribution significance of motif enrichment is computed; the cutoff optimizing this statistical criterion is selected for TFBS identification. For --feature_format DNAShape, shape motifs are collected from per-feature output directories: CNN/0 = HelT, CNN/1 = MGW, CNN/2 = ProT, CNN/3 = Roll. This removes subjective cutoff selection and was applied across the 690 human ENCODE ChIP-seq datasets (101-bp peaks) to report both sequence and shape motif counts.

## Related tools

- **DESSO** (Framework providing predict.py with binomial-distribution-based cutoff scanning for motif and TFBS identification from trained CNN models) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Backend used to train and run the CNN models whose predictions are scanned by the cutoff procedure)
- **Biopython** (Dependency for sequence/feature handling in the DESSO pipeline)
- **scikit-learn** (Dependency used in DESSO model training/prediction pipeline)

## Examples

```
python predict.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format DNAShape --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- AUC reported in Test_result.txt confirms the trained model predicts TF-DNA binding specificity before running predict.py.
- Motif/TFBS calls are produced for every cutoff in the scan (0.01 to 1 step 0.03) and the binomial-based optimal cutoff is selected rather than a fixed threshold.
- For DNAShape runs, exactly four shape motif sets appear (CNN/0–3 corresponding to HelT, MGW, ProT, Roll).
- Reproduction check: totals across 690 ENCODE datasets should match 1257 HelT, 84 MGW, 885 ProT, 478 Roll motifs, with 598/690 datasets having at least one shape motif.

## Limitations

- Sequence and DNA shape features covary, so the role of DNA shape in explaining TF-DNA binding is still debated.
- DL-based motif prediction is intrinsically limited by the availability of large-scale sequencing and protein-DNA interaction data such as ChIP-seq.
- Existing DL-based methods (including this one) assume each position in a motif detector contributes independently, which may underestimate nucleotide dependencies.
- Repository example data only contain wgEncodeEH002288-related files; the full ~5.9GB bundle must be downloaded for all 690 datasets.

## Evidence

- [intro] binomial distribution integrated into DESSO for TFBS identification: "we integrated the binomial distribution into DESSO to optimize the TFBSs identification based on identified motif patterns"
- [readme] predict.py cutoff scan parameters: "python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03"
- [readme] shape motif output directories: "For `--feature_format DNAShape`, four kinds of shape motifs would be predicted as shown in the following table"
- [abstract] binomial distribution model in DESSO design: "we developed a new method (DEep Sequence and Shape mOtif or DESSO) for cis-regulatory motif prediction using deep neural networks and the binomial distribution model"
- [abstract] shape predictive power finding: "We found that shape information has strong predictive power for TF–DNA binding and provides new putative shape motif information for human TFs"
