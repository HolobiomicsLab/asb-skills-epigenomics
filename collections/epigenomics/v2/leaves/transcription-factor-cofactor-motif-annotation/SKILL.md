---
name: transcription-factor-cofactor-motif-annotation
description: Use when you have DESSO motifs/TFBSs and ENCODE ChIP-seq peaks for multiple
  TFs in one cell line and need to classify indirect binding and infer TF-cofactor
  tethering from motif co-occurrence.
when_to_use_negative:
- You only need de novo motif discovery from ChIP-seq peaks without classifying direct
  vs indirect binding or inferring tethering
- ChIP-seq data lacks the paired motif/TFBS predictions needed to distinguish D, I,
  and K peaks
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DESSO
  role: Deep learning (CNN) framework used to predict sequence motifs and TFBSs from
    ChIP-seq peaks, which drive the D/I/K peak classification and tethering inference
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:e432f6cef4e6d131fc236438d7c2dae403a2d5fe3ddd7063fd38990fe44f2fb1
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
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# transcription-factor-cofactor-motif-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates ChIP-seq peaks of a TF as direct-binding (D), indirect-binding (I) or indecipherable (K) using DESSO-predicted motifs, then cross-matches I peaks against D peaks of other TFs to infer putative protein-protein-DNA tethering interactions (e.g. 61 associations among 100 K562 TFs, including ATF3-USF1 and NFE2-MAX).

## When to use

Use when you have DESSO motifs/TFBSs and ENCODE ChIP-seq peaks for multiple TFs in one cell line and need to classify indirect binding and infer TF-cofactor tethering from motif co-occurrence.

## When NOT to use

- You only need de novo motif discovery from ChIP-seq peaks without classifying direct vs indirect binding or inferring tethering
- ChIP-seq data lacks the paired motif/TFBS predictions needed to distinguish D, I, and K peaks

## Inputs

- DESSO-predicted sequence motifs and TFBSs for each TF
- ENCODE ChIP-seq peak files (BED) for the TFs expressed in the cell line (e.g. 100 K562 TFs)

## Outputs

- Per-TF peak classification into D (direct), I (indirect), and K (indecipherable) sets
- List of significant TF-TF tethering associations (Table S6 / tethering_associations.csv), e.g. 61 putative interactions including ATF3-USF1 and NFE2-MAX

## How to apply

Load the DESSO-identified sequence motifs and TFBSs predicted for each TF together with its ENCODE ChIP-seq peak files. Classify every peak of each TF into direct-binding (D), indirect-binding (I) or indecipherable (K) sets according to whether the peak contains the TF's own DESSO motif, another TF's motif, or neither, following Supplementary Method S5. Cross-match I peaks of each TF against D peaks of every other TF to count co-occurrences between absent-motif peaks and other TFs' motif-bearing peaks. Apply the statistical enrichment test from Supplementary Method S5 to each TF-TF pair to call significant tethering candidates, compile them (e.g. into tethering_associations.csv / Table S6), and report the putative protein-protein-DNA associations.

## Related tools

- **DESSO** (Deep learning (CNN) framework used to predict sequence motifs and TFBSs from ChIP-seq peaks, which drive the D/I/K peak classification and tethering inference) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The recovered association list contains exactly 61 putative tethering interactions among the 100 K562 TFs
- The association list includes the known tethering mechanisms ATF3-USF1 and NFE2-MAX
- Each peak is assigned to exactly one of D, I, or K based on own-motif, other-TF motif, or neither
- Only TF-TF pairs passing the Supplementary Method S5 statistical enrichment test are recorded as tethering candidates

## Limitations

- Predicted indirect TF-motif interaction mechanisms and shape contributions await experimental validation
- Inferred tethering candidates are statistical associations from motif co-occurrence, not experimentally confirmed protein-protein-DNA interactions

## Evidence

- [supplementary] Classify peaks into D/I/K sets per Supplementary Method S5: [evidence span withheld — anchor status: not_found]
