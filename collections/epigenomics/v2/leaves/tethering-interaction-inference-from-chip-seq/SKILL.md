---
name: tethering-interaction-inference-from-chip-seq
description: Use when ChIP-seq peak files and DESSO-predicted motifs/TFBSs are available
  for a panel of TFs in one cell type, and you need to identify co-factor-mediated
  (tethering) binding by finding peaks lacking a TF's own motif that co-occur with
  another TF's motif-bearing peaks.
when_to_use_negative:
- You only have peak files without DESSO-predicted motifs/TFBSs, since D/I/K peak
  classification depends on matching peaks against predicted motif instances
- The goal is direct motif discovery or DNA-shape motif prediction from ChIP-seq peaks
  rather than inferring co-factor-mediated tethering between TFs
- Only a single TF's ChIP-seq data is available, since the method requires cross-matching
  I peaks of one TF against D peaks of other TFs
tools:
- name: DESSO
  role: Deep learning (CNN) framework used to predict sequence motifs and TFBSs from
    ENCODE ChIP-seq peaks, which are required for classifying peaks into D/I/K sets
    and for the subsequent tethering cross-match
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:d7dfdf7c80b6148c12313923954302197653058c0f51f303f7e158c7ebc84f7b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# tethering-interaction-inference-from-chip-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers putative protein-protein-DNA tethering interactions by cross-matching DESSO-classified indirect-binding (I) peaks of one TF against direct-binding (D) peaks of another TF, applied here to the 100 TFs expressed in the K562 cell line.

## When to use

Use when ChIP-seq peak files and DESSO-predicted motifs/TFBSs are available for a panel of TFs in one cell type, and you need to identify co-factor-mediated (tethering) binding by finding peaks lacking a TF's own motif that co-occur with another TF's motif-bearing peaks.

## When NOT to use

- You only have peak files without DESSO-predicted motifs/TFBSs, since D/I/K peak classification depends on matching peaks against predicted motif instances
- The goal is direct motif discovery or DNA-shape motif prediction from ChIP-seq peaks rather than inferring co-factor-mediated tethering between TFs
- Only a single TF's ChIP-seq data is available, since the method requires cross-matching I peaks of one TF against D peaks of other TFs

## Inputs

- DESSO-predicted sequence motifs and TFBSs for each TF (from DESSO CNN runs on ENCODE ChIP-seq peaks)
- ENCODE ChIP-seq peak files (narrowPeak/BED) for the 100 TFs expressed in K562
- DESSO-trained CNN models per dataset (e.g., output/encode_101/gc_match/<dataset>/Seq/CNN)

## Outputs

- Peak classification per TF into D (direct), I (indirect), and K (indecipherable) sets
- Table of statistically enriched TF-TF tethering candidate pairs (Table S6-style)
- tethering_associations.csv listing putative tethering interactions (61 pairs for K562, including ATF3-USF1 and NFE2-MAX)

## How to apply

Load the DESSO-identified sequence motifs and TFBSs predicted for each TF together with its ENCODE ChIP-seq peak files, then classify every peak of each TF into direct-binding (D, contains the TF's own motif), indirect-binding (I, contains another TF's motif), or indecipherable (K, contains neither) sets following Supplementary Method S5. Cross-match the I peaks of each TF against the D peaks of every other TF to count co-occurrences between motif-absent peaks and other TFs' motif-bearing peaks. Apply the statistical enrichment test specified in Supplementary Method S5 to each TF-TF pair to call significant tethering candidates. Record significant pairs (e.g., as in Table S6) and compile them into a result table such as tethering_associations.csv. For the 100 K562 TFs this procedure recovered 61 putative tethering interactions, including the known ATF3-USF1 and NFE2-MAX pairs.

## Related tools

- **DESSO** (Deep learning (CNN) framework used to predict sequence motifs and TFBSs from ENCODE ChIP-seq peaks, which are required for classifying peaks into D/I/K sets and for the subsequent tethering cross-match) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The recovered association list contains exactly 61 putative tethering interactions among the 100 K562 TFs
- The association list includes the two known tethering mechanisms ATF3-USF1 and NFE2-MAX
- Every reported pair is supported by a significant result from the Supplementary Method S5 statistical enrichment test
- Each TF's peaks are partitioned into D, I, and K sets with every peak assigned to exactly one class based on own-motif, other-TF-motif, or no-motif presence
- All 100 K562-expressed TFs are processed and cross-matched pairwise (each TF's I peaks tested against every other TF's D peaks)

## Limitations

- Predicted indirect TF-motif interaction mechanisms inferred via tethering activities and co-factor motifs await experimental validation
- The approach depends on DESSO motif detection quality; peaks with neither the TF's own nor another TF's motif are indecipherable (K) and cannot contribute to tethering inference
- DESSO was trained/evaluated on human ENCODE ChIP-seq data (690 datasets) with dependencies such as TensorFlow 1.1.0, CUDA 8.0.44, and Python 2.7, which may constrain reuse in other environments

## Evidence

- [supplementary] workflow card describing peak classification and cross-matching: [evidence span withheld — anchor status: not_found]
