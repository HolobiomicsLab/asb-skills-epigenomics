---
name: aggregate-peak-analysis-zscore
description: Use when comparing CTCF-mediated loop callers (e.g., GILoop vs. Peakachu)
  on Hi-C contact maps and you need a threshold-independent visual/quantitative check
  that predicted loops are enriched in real contact signal, via APA z-scores computed
  around each caller's predicted loops.
when_to_use_negative:
- Input is extremely sparse single-cell Hi-C (scHi-C) data — the article states GILoop
  'only focuses on bulk Hi-C' and is not capable of dealing with scHi-C
- You need absolute loop calls rather than a comparative quality assessment of existing
  caller predictions (APA here is a benchmarking readout, not a caller)
- The targeted cell line's sequencing depth differs from all pre-trained models in
  the repository, where the article warns of 'weak usability'
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GILoop
  role: Primary loop caller under evaluation; its fused dual-branch model produces
    genome-wide CTCF-mediated loop probability maps on the 520M-read matrices that
    are assessed by APA z-score
  repo: github.com/fzbio/GILoop
- name: Peakachu
  role: Comparison baseline; run in both full (clustering) and non-clustering modes
    on the same 520M-read matrices to obtain predictions assessed by APA z-score alongside
    GILoop
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.105535
    title: 'GILoop: Robust chromatin loop calling across multiple sequencing depths
      on Hi-C data'
  source_content_hash: sha256:d59e0e71ac1b098770352bff2b47eac9348cc54c0be046e72a2fce0643302e1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GILoop
  - Peakachu
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# aggregate-peak-analysis-zscore

## Summary

Aggregate peak analysis (APA) with z-score enrichment to quantify whether a chromatin loop caller's predicted loops concentrate Hi-C contact signal at the loop center, used alongside PR curves and well-supported-loop recovery to benchmark GILoop against Peakachu (full and non-clustering) on the 520 million-read GM12878 Hi-C dataset.

## When to use

Use when comparing CTCF-mediated loop callers (e.g., GILoop vs. Peakachu) on Hi-C contact maps and you need a threshold-independent visual/quantitative check that predicted loops are enriched in real contact signal, via APA z-scores computed around each caller's predicted loops.

## When NOT to use

- Input is extremely sparse single-cell Hi-C (scHi-C) data — the article states GILoop 'only focuses on bulk Hi-C' and is not capable of dealing with scHi-C
- You need absolute loop calls rather than a comparative quality assessment of existing caller predictions (APA here is a benchmarking readout, not a caller)
- The targeted cell line's sequencing depth differs from all pre-trained models in the repository, where the article warns of 'weak usability'

## Inputs

- Hi-C contact maps at the 520 million-read GM12878 sequencing depth (.cool files or Juicer-dump KR-normalized O/E text matrices)
- Genome-wide loop probability/loop predictions from GILoop (fused dual-branch model)
- Loop predictions from Peakachu full (clustering) mode and non-clustering mode
- Ground-truth loop set defined as the intersection of HICCUPS and FitHiC calls

## Outputs

- APA z-score enrichment values per caller (GILoop, Peakachu full, Peakachu non-clustering)
- Precision-recall curves per caller against the HICCUPS∩FitHiC ground-truth loop set
- Well-supported-loop recovery fractions across score thresholds
- Combined benchmark figure (figure3_model_comparison.png)

## How to apply

Run each caller (the fused GILoop model in inference mode, plus Peakachu in both its full clustering mode and its non-clustering mode) on the same KR-normalized O/E matrices, e.g., the 520 million-read GM12878 Hi-C data. Sweep each caller's score threshold and compute precision and recall against the ground-truth loop set constructed as the intersection of HICCUPS and FitHiC calls. For each caller's predicted loop set, perform aggregate peak analysis (APA) by stacking local contact-map patches centered on predicted loops and compute the APA z-score enrichment. Quantify well-supported-loop recovery as the fraction of high-confidence ground-truth loops recovered by each caller across score thresholds. Combine the PR curves, APA z-score enrichment, and recovery panels into a single benchmark figure (e.g., figure3_model_comparison.png).

## Related tools

- **GILoop** (Primary loop caller under evaluation; its fused dual-branch model produces genome-wide CTCF-mediated loop probability maps on the 520M-read matrices that are assessed by APA z-score) — github.com/fzbio/GILoop
- **Peakachu** (Comparison baseline; run in both full (clustering) and non-clustering modes on the same 520M-read matrices to obtain predictions assessed by APA z-score alongside GILoop)

## Examples

```
python demo.py
```

## Evaluation signals

- GILoop's PR-AUC exceeds Peakachu's on the 520 million-read test sets across all probability thresholds, consistent with 'significantly higher PR-AUC'
- APA z-score enrichment around GILoop's predicted loops matches or exceeds that of Peakachu full and non-clustering variants
- Well-supported-loop recovery curves show the expected fraction of HICCUPS∩FitHiC high-confidence loops recovered at each score threshold
- All three panels (PR curves, APA z-score, recovery) are rendered together in the saved combined figure (figure3_model_comparison.png)
- Input matrices are confirmed KR-normalized O/E when using the Juicer-dump input path, since the script applies no preprocessing

## Limitations

- GILoop requires substantial pre-processing time before training/inference, mainly from the downsampling procedure (matrix transformations and hard disk reading/writing)
- Weak usability if the targeted cell line's sequencing depth differs from any pre-trained model provided in the repository
- GILoop only focuses on bulk Hi-C and cannot deal with extremely sparse single-cell Hi-C data

## Evidence

- [discussion] GILoop significantly outperforms Peakachu in finding CTCF-associated loops – the latter is the most recent machine learning-based chromatin loop detector: "GILoop significantly outperforms Peakachu in finding CTCF-associated loops – the latter is the most recent machine learning-based chromatin loop detector"
- [abstract] Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops.: "Here we propose GILoop, a dual-branch neural network that learns from both representations to identify genome-wide CTCF-mediated loops."
- [readme] This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.): "This script is the full pipeline including data pre-processing (sequencing depth alignment, normalization, and expected vector calculation, etc.), and GILoop sampling and training."
- [readme] please make sure the text files are KR-normalized O/E matrices: "In this case, please make sure the text files are KR-normalized O/E matrices, and the source and the target are of similar number of reads."
