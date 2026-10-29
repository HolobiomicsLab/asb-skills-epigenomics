---
name: loop-benchmark-against-hic-reference-datasets
description: Use when a newly called loop list from chromatin tracing/imaging data
  must be validated against HiCCUPS loop anchors from bulk Hi-C at matched resolution
  (e.g., 10 Kb or 25 Kb), or when precision/recall/F1 of a loop caller are needed.
when_to_use_negative:
- The imaging dataset is not mESC or no deeply sequenced bulk Hi-C reference loop
  set exists at a matching resolution — overlap-based precision/recall would be uninterpretable.
- The goal is de novo loop calling rather than benchmarking — use 'SnapFISH call-loops'
  directly; this skill only evaluates an already-produced loop list.
- Comparing against another imaging-based loop set at the same locus (e.g., Sox2 enhancer-promoter
  overlap within 10 Kb) where count-based Hi-C references do not apply.
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3169
tools:
- name: SnapFISH
  role: Produces the imaging-derived loop summit list that is benchmarked; called
    via 'SnapFISH call-loops' with FDR threshold -h 0.1
  repo: https://github.com/HuMingLab/SnapFISH
- name: HiCCUPS
  role: Provides the reference loop anchor set from deeply sequenced mESC bulk Hi-C
    used as ground truth for overlap counting
- name: pandas
  role: Python dependency (v1.1.5) for loading loop lists, counting anchor overlaps,
    and writing the tab-separated results table
- name: numpy
  role: Python dependency (v1.19.5) supporting distance/overlap computation during
    benchmarking
- name: scipy
  role: Python dependency (v1.5.4) supporting statistical computations in the upstream
    SnapFISH pipeline whose output is benchmarked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40658-3
    title: 'SnapFISH: a computational pipeline to identify chromatin loops from multiplexed
      DNA FISH data'
  source_content_hash: sha256:252adbc2dd358fd64fa8259fa1d869c97a9bb624e3b835a298f5eabee9ecd3b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapFISH
  tools:
  - HiCCUPS
  - Python
  - SnapFISH
  - matplotlib
  - numpy
  - pandas
  - scipy
  - statsmodels
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# loop-benchmark-against-hic-reference-datasets

## Summary

Benchmarks imaging-derived chromatin loops (e.g., SnapFISH loops from multiplexed DNA FISH data) against a proximity-ligation reference loop set (HiCCUPS calls from deeply sequenced mESC bulk Hi-C) by counting anchor overlaps and computing precision, recall, and F1-score.

## When to use

Use when a newly called loop list from chromatin tracing/imaging data must be validated against HiCCUPS loop anchors from bulk Hi-C at matched resolution (e.g., 10 Kb or 25 Kb), or when precision/recall/F1 of a loop caller are needed.

## When NOT to use

- The imaging dataset is not mESC or no deeply sequenced bulk Hi-C reference loop set exists at a matching resolution — overlap-based precision/recall would be uninterpretable.
- The goal is de novo loop calling rather than benchmarking — use 'SnapFISH call-loops' directly; this skill only evaluates an already-produced loop list.
- Comparing against another imaging-based loop set at the same locus (e.g., Sox2 enhancer-promoter overlap within 10 Kb) where count-based Hi-C references do not apply.

## Inputs

- SnapFISH loop summit list (e.g., output_loop_summit_SUF.txt from 'SnapFISH call-loops')
- HiCCUPS loop anchor list from deeply sequenced mESC bulk Hi-C at 10 Kb and/or 25 Kb resolution
- 1D genomic annotation file (ANN.txt: chr, start, end, pos) for coordinate mapping
- Imaging dataset metadata (e.g., 446 cells / 892 alleles of mESC DNA seqFISH+ at 25 Kb resolution)

## Outputs

- Overlap count between imaging-derived loops and HiCCUPS reference loops
- Precision, recall, and F1-score metrics (target: 16 loops, precision = 0.875, recall = 0.400, F1 = 0.549)
- Tab-separated results table containing loop summits and precision/recall/F1 fields

## How to apply

Run the loop caller (SnapFISH 'call-loops' with FDR < 0.1, retaining cluster summits with population-level contact frequency >= 1/3 and singletons with frequency >= 1/2) to obtain the imaging-based loop summit list. Download HiCCUPS loop anchors from deeply sequenced mESC bulk Hi-C, processed at 10 Kb and/or 25 Kb resolution to match the imaging bin size. Count overlaps between SnapFISH loop anchors and HiCCUPS loop anchors using a defined distance tolerance (e.g., anchors within 10 Kb, as used for Sox2 enhancer-promoter comparisons in the paper). Compute precision = overlapping SnapFISH loops / total SnapFISH loops, recall = overlapping SnapFISH loops / total HiCCUPS loops, and F1 = 2PR/(P+R). For the mESC DNA seqFISH+ benchmark (446 cells, 892 alleles at 25 Kb), the expected result is 16 loops, 14 overlapping HiCCUPS loops, precision = 87.5%, recall = 40.0%, F1 = 0.549. Write the summit list and metrics to a tab-separated results table and validate fields against expected values.

## Related tools

- **SnapFISH** (Produces the imaging-derived loop summit list that is benchmarked; called via 'SnapFISH call-loops' with FDR threshold -h 0.1) — https://github.com/HuMingLab/SnapFISH
- **HiCCUPS** (Provides the reference loop anchor set from deeply sequenced mESC bulk Hi-C used as ground truth for overlap counting)
- **pandas** (Python dependency (v1.1.5) for loading loop lists, counting anchor overlaps, and writing the tab-separated results table)
- **numpy** (Python dependency (v1.19.5) supporting distance/overlap computation during benchmarking)
- **scipy** (Python dependency (v1.5.4) supporting statistical computations in the upstream SnapFISH pipeline whose output is benchmarked)

## Examples

```
SnapFISH call-loops -i PATH/TO/3DCOOR.txt -o OUT/DIR -a PATH/TO/ANN.txt -p 1 -d SUFFIX
```

## Evaluation signals

- Output file exists and contains precision, recall, and F1 fields matching expected values: 16 loops, precision = 0.875, recall = 0.400, F1 = 0.549
- Overlap count between SnapFISH loops and HiCCUPS anchors equals 14 of the 16 called loops
- Precision and recall are consistent: precision = overlaps/16 = 0.875 and recall = overlaps/|HiCCUPS loops| = 0.400 imply a consistent reference loop count
- F1 recomputes as 2*P*R/(P+R) ≈ 0.549 from the reported precision and recall
- Benchmark resolution of the HiCCUPS reference (10 Kb or 25 Kb) matches the imaging bin resolution used for loop calling

## Limitations

- Sensitivity of SnapFISH can be further improved, particularly for DNA seqFISH+ data in mESCs — low recall (40.0%) partly reflects the reference and data limitations.
- Imputation of missing 3D coordinates/distances is proposed but not yet implemented, which may limit statistical power and thus measured recall.
- Only pair-wise chromatin interactions are considered; multi-way interactions are not detected or benchmarked.

## Evidence

- [methods] FDR filtering threshold (-h): "-h 0.1: The filtering threshold of the FDR. Only bin pairs with the FDR less than the threshold will be kept. Default setting is 0.1."
- [discussion] Final loop contact frequency thresholds: "The final loop list consists of cluster summits with population-level contact frequency >=1/3, and singletons with population-level contact frequency >=1/2"
- [discussion] Anchor overlap distance tolerance: "if and only if the SnapFISH loop anchors are within 10 Kb of Sox2 promoter-super-enhancer loop anchors"
- [readme] Command-line invocation: "SnapFISH call-loops -i PATH/TO/3DCOOR.txt -o OUT/DIR -a PATH/TO/ANN.txt -p 1 -d SUFFIX"
- [discussion] Loop candidate definition: "(T < 0) and FDR < 10%"
