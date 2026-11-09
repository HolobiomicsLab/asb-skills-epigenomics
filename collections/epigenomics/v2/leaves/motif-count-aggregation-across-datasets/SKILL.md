---
name: motif-count-aggregation-across-datasets
description: Use when DESSO shape-motif prediction outputs (DNAShape/CNN/0-3 per-dataset
  directories) must be tallied into per-shape-type totals and per-dataset presence/absence
  summaries across many ChIP-seq datasets.
when_to_use_negative:
- Analyzing single-dataset motif results where aggregation across the 690-dataset
  corpus is not needed
- Counting sequence motifs from --feature_format Seq runs, which output to a single
  CNN/0 directory and are not split by shape type
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0749
tools:
- name: DESSO
  role: CNN-based deep learning framework whose DNAShape predict.py outputs (binomial-distribution
    cutoff scan) supply the per-dataset shape motifs being tallied
  repo: https://github.com/OSU-BMBL/DESSO
- name: TensorFlow
  role: Dependency used by DESSO to train and run the CNN shape-feature models whose
    motif outputs are aggregated
- name: Biopython
  role: Python dependency of the DESSO pipeline used in processing the motif prediction
    outputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:6e9ba3a7070610023a8525b50398daff2557743372d44753f11c3a9a43469429
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# motif-count-aggregation-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates per-dataset motif predictions from DESSO's CNN/DNAShape runs across all 690 human ENCODE ChIP-seq datasets, tallying HelT, MGW, ProT, and Roll shape motifs and the number of datasets with at least one shape motif, to test whether the human genome contains conserved DNA shape motifs discoverable like sequence motifs.

## When to use

Use when DESSO shape-motif prediction outputs (DNAShape/CNN/0-3 per-dataset directories) must be tallied into per-shape-type totals and per-dataset presence/absence summaries across many ChIP-seq datasets.

## When NOT to use

- Analyzing single-dataset motif results where aggregation across the 690-dataset corpus is not needed
- Counting sequence motifs from --feature_format Seq runs, which output to a single CNN/0 directory and are not split by shape type

## Inputs

- DESSO per-dataset shape motif output directories output/encode_101/gc_match/<dataset>/DNAShape/CNN/0-3 (HelT, MGW, ProT, Roll)
- List of the 690 ENCODE ChIP-seq dataset identifiers (e.g. wgEncodeEH002288)
- DESSO shape motif prediction results generated with --feature_format DNAShape --peak_flank 50

## Outputs

- shape_motif_counts.csv with per-shape-type motif totals and count of datasets with at least one shape motif
- Per-dataset, per-shape-type presence/absence tally

## How to apply

After running DESSO's predict.py with --feature_format DNAShape and the binomial-distribution cutoff scan (--start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03) on the 101-bp peak regions (peak_flank 50), collect predicted shape motifs from the four per-feature output directories: CNN/0 = HelT, CNN/1 = MGW, CNN/2 = ProT, CNN/3 = Roll. For each of the 690 ENCODE datasets, record whether at least one motif of each shape type was predicted, then aggregate total motif counts per shape type across all datasets. Write a summary table (e.g. shape_motif_counts.csv) containing per-shape-type counts and the number of datasets with at least one shape motif. Validate against the published reference totals of 1257 HelT, 84 MGW, 885 ProT, and 478 Roll motifs, with 598 of 690 datasets having at least one shape motif.

## Related tools

- **DESSO** (CNN-based deep learning framework whose DNAShape predict.py outputs (binomial-distribution cutoff scan) supply the per-dataset shape motifs being tallied) — https://github.com/OSU-BMBL/DESSO
- **TensorFlow** (Dependency used by DESSO to train and run the CNN shape-feature models whose motif outputs are aggregated)
- **Biopython** (Python dependency of the DESSO pipeline used in processing the motif prediction outputs)

## Examples

```
python predict.py --start_index 0 --end_index 690 --peak_flank 50 --network CNN --feature_format DNAShape --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Reproduced totals must match the reported counts: 1257 HelT, 84 MGW, 885 ProT, 478 Roll motifs
- Exactly 598 of the 690 ENCODE datasets must have at least one shape motif
- Each dataset must contribute at most one counted motif per shape-type directory (CNN/0=HelT, CNN/1=MGW, CNN/2=ProT, CNN/3=Roll)
- Summary table shape_motif_counts.csv must contain both per-shape-type counts and the dataset-with-motif count
- All 690 dataset indices (start_index/end_index ranges) must be covered with no missing wgEncodeEH* identifiers

## Limitations

- Counts depend on the binomial-distribution-based cutoff scan parameters (--start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03); changing these changes motif totals
- Requires the full ~5.9GB DESSO data bundle since the public GitHub repo ships only wgEncodeEH002288 example data
- The role of DNA shape in explaining TF-DNA binding remains debated because sequence and shape features covary

## Evidence
