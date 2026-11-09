---
name: protein-protein-dna-association-enrichment-testing
description: Use when you have DESSO motif/TFBS predictions and ENCODE ChIP-seq peak
  files for multiple TFs in one cell line (e.g. the 100 K562 TFs) and want to identify
  indirect (tethering) TF-DNA binding mechanisms.
when_to_use_negative:
- The input is a single TF's ChIP-seq dataset with no other TFs' motifs or peaks available
  for cross-matching, so I-peak vs D-peak co-occurrence cannot be tested
- The goal is de novo motif discovery or TF-DNA binding specificity prediction rather
  than inferring indirect/tethering binding mechanisms
- Peaks lack motif annotations (no DESSO motif/TFBS output), so D/I/K classification
  per Supplementary Method S5 cannot be performed
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0797
tools:
- name: DESSO
  role: Deep-learning CNN framework used to predict each TF's sequence motif and TFBSs,
    which define the D/I/K peak classification underlying the tethering cross-match
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:745881f9c918785ff4e0925fdb43bf3c06a666d891dd5231b1f526071af34d32
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
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# protein-protein-dna-association-enrichment-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers putative protein-protein-DNA tethering interactions by classifying ChIP-seq peaks into direct-binding (D), indirect-binding (I), and indecipherable (K) sets using DESSO-predicted motifs, then statistically testing co-occurrence of one TF's I peaks with another TF's D peaks. In the DESSO paper this recovered 61 putative tethering associations among the 100 TFs expressed in K562.

## When to use

Use when you have DESSO motif/TFBS predictions and ENCODE ChIP-seq peak files for multiple TFs in one cell line (e.g. the 100 K562 TFs) and want to identify indirect (tethering) TF-DNA binding mechanisms.

## When NOT to use

- The input is a single TF's ChIP-seq dataset with no other TFs' motifs or peaks available for cross-matching, so I-peak vs D-peak co-occurrence cannot be tested
- The goal is de novo motif discovery or TF-DNA binding specificity prediction rather than inferring indirect/tethering binding mechanisms
- Peaks lack motif annotations (no DESSO motif/TFBS output), so D/I/K classification per Supplementary Method S5 cannot be performed

## Inputs

- DESSO-predicted sequence motifs and TFBSs for the 100 TFs expressed in K562
- ENCODE ChIP-seq peak files (BED) for the 100 K562 TFs
- Reference genome GRCh37.p13.genome.fa (for DESSO sequence extraction)
- TF expression/tfbs annotations from ENCODE (e.g. TfbsUniform_hg19_ENCODE)

## Outputs

- Per-TF peak classification into D (direct), I (indirect), and K (indecipherable) sets
- Statistically enriched TF-TF tethering candidate pairs (Table S6)
- tethering_associations.csv listing the 61 putative protein-protein-DNA tethering interactions

## How to apply

First run DESSO (CNN, Seq feature format, peak_flank 50, cutoff sweep start 0.01, end 1, step 0.03) to obtain each TF's sequence motif and TFBSs, then classify every ChIP-seq peak per TF into D (contains the TF's own DESSO motif), I (contains another TF's motif but not its own), or K (neither), per Supplementary Method S5. Cross-match each TF's I peaks against the D peaks of every other TF among the 100 K562 TFs, counting co-occurrences between motif-absent peaks and the other TF's motif-bearing peaks. Apply the statistical enrichment test specified in Supplementary Method S5 to each TF-TF pair to call significant tethering candidates (recorded in Table S6). Compile significant pairs into tethering_associations.csv; the reference result is 61 putative tethering interactions including the known ATF3-USF1 and NFE2-MAX pairs.

## Related tools

- **DESSO** (Deep-learning CNN framework used to predict each TF's sequence motif and TFBSs, which define the D/I/K peak classification underlying the tethering cross-match) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- The recovered association list contains exactly 61 putative tethering interactions among the 100 K562 TFs
- The list includes the two known tethering mechanisms ATF3-USF1 and NFE2-MAX
- Every peak of each TF is assigned to exactly one of the D, I, or K sets per Supplementary Method S5
- Each reported TF-TF pair passes the statistical enrichment test specified in Supplementary Method S5
- Predicted tethering pairs arise from co-occurrence of a TF's I peaks with the other TF's motif-bearing D peaks, not from D-D overlap

## Limitations

- Predicted indirect TF-motif interaction mechanisms and shape contributions await experimental validation
- The tethering test depends on DESSO motif-detection quality; peaks without any recognizable motif are relegated to the indecipherable (K) set and excluded
- DESSO application is intrinsically limited by availability of large-scale ChIP-seq data, since such experiments are expensive

## Evidence

- [results] Cross-matching I peaks of one TF against D peaks of another among the 100 K562 TFs, DESSO predicted 61 putative tethering associations: [evidence span withheld — anchor status: not_found]
