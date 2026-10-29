---
name: tfbs-cross-matching-between-transcription-factors
description: Use when ChIP-seq peaks and DESSO-predicted TFBSs are available for multiple
  TFs in one cell line (e.g., K562) and the goal is to identify TF pairs where one
  binds DNA only via tethering to the other's motif-bearing peak.
when_to_use_negative:
- Only a single TF's ChIP-seq data is available — cross-matching requires at least
  two TFs' peak and motif sets in the same cell line
- The goal is de novo motif discovery or TFBS identification itself rather than inferring
  TF-TF tethering relationships from already-predicted TFBSs
- Peak files lack summit coordinates or cannot be intersected by genomic coordinates,
  so I/D peak overlap cannot be computed
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: DESSO
  role: Deep learning (CNN) framework used to predict sequence motifs and identify
    TFBSs in ChIP-seq peaks, which supply the motif presence/absence calls used to
    classify peaks as D, I, or K
  repo: https://github.com/OSU-BMBL/DESSO
- name: bedtools
  role: Used in the DESSO pipeline to extract DNA sequences from peak files, supporting
    motif-based peak classification
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:a24d0b77df02edd83c3f89045d5d19ce5ad0459821c80a002d8e23db33313a7e
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
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tfbs-cross-matching-between-transcription-factors

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers putative protein-protein-DNA tethering interactions by cross-matching indirect-binding (I) ChIP-seq peaks of one TF against direct-binding (D) peaks of another, using DESSO-predicted motifs and a statistical enrichment test. In the source study this recovered 61 putative tethering associations among the 100 TFs expressed in K562, including ATF3-USF1 and NFE2-MAX.

## When to use

Use when ChIP-seq peaks and DESSO-predicted TFBSs are available for multiple TFs in one cell line (e.g., K562) and the goal is to identify TF pairs where one binds DNA only via tethering to the other's motif-bearing peak.

## When NOT to use

- Only a single TF's ChIP-seq data is available — cross-matching requires at least two TFs' peak and motif sets in the same cell line
- The goal is de novo motif discovery or TFBS identification itself rather than inferring TF-TF tethering relationships from already-predicted TFBSs
- Peak files lack summit coordinates or cannot be intersected by genomic coordinates, so I/D peak overlap cannot be computed

## Inputs

- DESSO-predicted sequence motifs and TFBSs for each TF (CNN motif prediction output, e.g., output/encode_101/gc_match/<dataset>/Seq/CNN/0)
- ENCODE ChIP-seq peak files (BED/narrowPeak) for the TFs expressed in the cell line of interest (e.g., 100 K562 TFs)
- TF expression/cell-line assignment list to restrict analysis to TFs expressed in the target cell line

## Outputs

- Per-TF peak classification into direct-binding (D), indirect-binding (I), and indecipherable (K) peak sets
- TF-TF pairs with co-occurrence counts between I peaks and D peaks
- tethering_associations.csv listing significant putative tethering interactions (e.g., 61 pairs, including ATF3-USF1 and NFE2-MAX)

## How to apply

Load the DESSO-identified sequence motifs and TFBSs predicted for each TF (e.g., the 100 TFs expressed in K562) together with their ENCODE ChIP-seq peak files. Classify every peak of each TF into direct-binding (D), indirect-binding (I), or indecipherable (K) sets according to whether the peak contains the TF's own DESSO motif, another TF's motif, or neither (Supplementary Method S5). Cross-match I peaks of each TF against D peaks of every other TF to count co-occurrences between absent-motif peaks and other TFs' motif-bearing peaks. Apply the statistical enrichment test from Supplementary Method S5 to each TF-TF pair to call significant tethering candidates, compile them into a table (e.g., tethering_associations.csv / Table S6), and validate against reported mechanisms such as ATF3-USF1 and NFE2-MAX. Rationale: peaks lacking the ChIPed TF's own motif but enriched for another TF's motif indicate binding via protein-protein tethering rather than direct sequence-specific DNA contact.

## Related tools

- **DESSO** (Deep learning (CNN) framework used to predict sequence motifs and identify TFBSs in ChIP-seq peaks, which supply the motif presence/absence calls used to classify peaks as D, I, or K) — https://github.com/OSU-BMBL/DESSO
- **bedtools** (Used in the DESSO pipeline to extract DNA sequences from peak files, supporting motif-based peak classification) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The recovered association list contains exactly 61 putative tethering interactions among the 100 K562 TFs
- The list includes the two known tethering mechanisms ATF3-USF1 and NFE2-MAX
- Each peak is assigned to exactly one of the D, I, or K sets per TF, with no unclassified peaks
- Each reported TF-TF pair passes the statistical enrichment test specified in Supplementary Method S5
- Output table matches the schema of Table S6 (TF-TF pair, co-occurrence counts, enrichment statistics)

## Limitations

- Predicted indirect TF-motif interaction mechanisms inferred via tethering activities and co-factor motifs await experimental validation
- Peak classification depends on DESSO motif quality; peaks containing neither the TF's own motif nor another TF's motif are indecipherable (K) and cannot inform tethering inference
- The enrichment test follows Supplementary Method S5; results depend on its significance threshold and the motif cutoff used by DESSO's binomial-distribution TFBS identification

## Evidence

- [full_text] Cross-matching I peaks of one TF against D peaks of another among the 100 K562 TFs, DESSO predicted 61 putative tethering associations: [evidence span withheld — anchor status: not_found]
