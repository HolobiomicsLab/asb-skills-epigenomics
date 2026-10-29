---
name: chip-seq-motif-prediction
description: Use when raw or processed human ChIP-seq peak calls (e.g. ENCODE BED
  with summits) are available and the goal is to discover TF-DNA binding motifs with
  significance scores and compare them to documented motifs.
when_to_use_negative:
- Input is already a curated motif or feature table rather than ChIP-seq peak coordinates;
  no motif discovery is needed.
- The organism/genome lacks a matched reference FASTA and GC-matched background sequences,
  so per-dataset CNN training cannot be grounded.
- Only DNA shape features (HelT, MGW, ProT, Roll) are of interest and no sequence-based
  model comparison is required — use the DNAShape branch specifically instead.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: DESSO
  role: CNN deep learning framework for motif prediction and TFBS identification via
    binomial-distribution cutoff optimization
  repo: https://github.com/OSU-BMBL/DESSO
- name: DeepBind
  role: Comparison method for motif prediction benchmarking on the same 690 ENCODE
    datasets
- name: Basset
  role: Comparison method for motif prediction benchmarking
- name: MEME-ChIP
  role: Comparison method for motif prediction benchmarking
- name: KMAC
  role: Comparison method for motif prediction benchmarking
- name: gkm-SVM
  role: Comparison method for motif prediction benchmarking
- name: TOMTOM
  role: All-against-all motif similarity search and clustering against JASPAR/TRANSFAC
    documented motifs
- name: TensorFlow
  role: Backend for CNN/GCNN model training and prediction
- name: bedtools
  role: Extracts DNA sequences from custom peak BED files before training
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:3e46c04222a49d3186a9cbe89ad4ba04fec9acd4dcaa564a724e9d125c8994cc
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
  - Basset
  - DESSO
  - DeepBind
  - KMAC
  - MEME-ChIP
  - TOMTOM
  - TensorFlow
  - bedtools
  - gkm-SVM
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# chip-seq-motif-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a CNN-based deep learning framework (DESSO) on 101-bp ChIP-seq peak sequences from ENCODE datasets and predict sequence/shape motifs with a binomial-distribution cutoff scan, benchmarked against DeepBind, Basset, MEME-ChIP, KMAC and gkm-SVM using TOMTOM similarity to JASPAR/TRANSFAC motifs.

## When to use

Use when raw or processed human ChIP-seq peak calls (e.g. ENCODE BED with summits) are available and the goal is to discover TF-DNA binding motifs with significance scores and compare them to documented motifs.

## When NOT to use

- Input is already a curated motif or feature table rather than ChIP-seq peak coordinates; no motif discovery is needed.
- The organism/genome lacks a matched reference FASTA and GC-matched background sequences, so per-dataset CNN training cannot be grounded.
- Only DNA shape features (HelT, MGW, ProT, Roll) are of interest and no sequence-based model comparison is required — use the DNAShape branch specifically instead.

## Inputs

- ENCODE ChIP-seq peak files (e.g. wgEncodeEH002288 BED with peak summits)
- Reference genome FASTA (GRCh37.p13.genome.fa)
- GC-matched background sequence sets (encode_101_background)
- Custom peak BED files (e.g. fox01_peaks.bed) with hg38.fa for own-data motif prediction
- Documented motif databases (JASPAR, TRANSFAC) for TOMTOM evaluation

## Outputs

- Trained CNN models under output/encode_101/gc_match/<dataset>/Seq/CNN with Test_result.txt AUC
- Predicted sequence motifs in output/encode_101/gc_match/<dataset>/Seq/CNN/0
- Shape motifs (HelT, MGW, ProT, Roll) under DNAShape/CNN/0-3
- TOMTOM similarity scores (E-value, P-value, q-value) versus JASPAR/TRANSFAC motifs
- Clustered representative motif summary (e.g. motif_cluster_summary.tsv) with known vs undocumented counts

## How to apply

Extract 101-bp sequences around each peak summit (peak_flank 50, i.e. 2*50+1 bp) from the ChIP-seq peaks using the GRCh37.p13 genome and GC-matched background sets (encode_101_background). Train a per-dataset CNN with train.py (--network CNN --feature_format Seq), recording test AUCs in Test_result.txt, then predict motifs with predict.py using the cutoff scan --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03; the binomial distribution replaces a subjective cutoff to optimize TFBS identification. Run comparison pipelines (DeepBind, Basset, MEME-ChIP, KMAC, gkm-SVM) on the same datasets and evaluate all predicted motifs against JASPAR/TRANSFAC documented motifs via TOMTOM. For shape motifs, retrain with --feature_format DNAShape to obtain four motif types (HelT, MGW, ProT, Roll); for 1001-bp long sequences use --network GCNN --peak_flank 500. Success is judged by DESSO's significantly larger -log2(E-value), -log2(P-value) and -log2(q-value) (Wilcoxon test P < 5e-4) versus the other tools, and by TOMTOM matches clustering representative motifs into known (435 of 683) versus undocumented (248) classes.

## Related tools

- **DESSO** (CNN deep learning framework for motif prediction and TFBS identification via binomial-distribution cutoff optimization) — https://github.com/OSU-BMBL/DESSO
- **DeepBind** (Comparison method for motif prediction benchmarking on the same 690 ENCODE datasets)
- **Basset** (Comparison method for motif prediction benchmarking)
- **MEME-ChIP** (Comparison method for motif prediction benchmarking)
- **KMAC** (Comparison method for motif prediction benchmarking)
- **gkm-SVM** (Comparison method for motif prediction benchmarking)
- **TOMTOM** (All-against-all motif similarity search and clustering against JASPAR/TRANSFAC documented motifs)
- **TensorFlow** (Backend for CNN/GCNN model training and prediction)
- **bedtools** (Extracts DNA sequences from custom peak BED files before training)

## Examples

```
python train.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq; python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Test_result.txt reports a per-dataset AUC for the trained CNN on held-out test data; low AUC indicates the model (and downstream motifs) are unreliable.
- Reproduced benchmark should show DESSO with significantly larger -log2(E-value), -log2(P-value) and -log2(q-value) than DeepBind, Basset, MEME-ChIP, KMAC and gkm-SVM (Wilcoxon test P < 5e-4, Figure 2A).
- TOMTOM clustering of representative motifs should reproduce the split of 683 representative motifs into 435 known (JASPAR/TRANSFAC-supported) and 248 undocumented motifs.
- Output directory structure must follow output/encode_101/gc_match/<dataset>/Seq/CNN/0 for sequence motifs and DNAShape/CNN/0-3 for the four shape motif types.
- Peak length check: peak_flank 50 must yield 2*50+1 = 101-bp sequences; peak_flank 500 must yield 1001-bp sequences for GCNN.

## Limitations

- Existing DL-based methods, including this framework, assume each position in a motif detector contributes to binding affinity independently, which may underestimate nucleotide dependencies.
- Application of DL to motif prediction is intrinsically limited by the availability of large-scale sequencing and protein-DNA interaction data such as ChIP-seq, and experiments are expensive.
- The role of DNA shape in explaining TF-DNA binding is still debated because sequence and shape features covary.
- Predicted indirect TF-motif interactions (tethering/co-factor motifs) and shape contributions await experimental validation.
- The GitHub repo ships only wgEncodeEH002288 example data due to file size limits; the full datasets (~5.9GB) require a separate download.

## Evidence

- [abstract] DESSO outperformed existing tools, including DeepBind, in predicting motifs in 690 human ENCODE ChIP-sequencing datasets: "DESSO outperformed existing tools, including DeepBind, in predicting motifs in 690 human ENCODE ChIP-sequencing datasets"
- [intro] TOMTOM evaluation databases: "The identified motifs were evaluated using the documented motifs in JASPAR (28) and TRANSFAC (29)"
