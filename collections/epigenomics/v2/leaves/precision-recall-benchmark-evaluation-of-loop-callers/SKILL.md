---
name: precision-recall-benchmark-evaluation-of-loop-callers
description: Use when comparing called chromatin loop sets from scHi-C loop-calling
  tools against a merged bulk-Hi-C/HiChIP reference list to rank methods by precision,
  recall, and F1 across subsampling levels.
when_to_use_negative:
- Input is already a scored metric table or publication-reported benchmark values
  rather than called loop bedpe sets to be re-evaluated
- The goal is calling loops on raw scHi-C contact files rather than evaluating previously
  called loop sets
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SnapHiC
  role: Loop-calling method under benchmark evaluation; supplies *.postprocessed.summits.bedpe
    loop summits from subsampled mES scHi-C data
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Used to merge reference loop sources, perform genomic interval overlap matching
    of called loops, and compute precision/recall/F1 into loop_calling_benchmark_f1.csv
- name: cooler
  role: Converts SnapHiC count matrices into .cool files used in the pipeline whose
    loop outputs are benchmarked
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:3fcbafd85dbbecbbb165b1620be6430bee14beec125bea3700e88c0363877260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  - cooler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# precision-recall-benchmark-evaluation-of-loop-callers

## Summary

Evaluates chromatin loop callers (e.g., SnapHiC, HiCCUPS, FastHiC, FitHiC2, HiC-ACT) by matching each tool's called loop summits (bedpe) against a merged reference loop list and computing per-method precision, recall, and harmonic-mean F1 across data subsampling levels.

## When to use

Use when comparing called chromatin loop sets from scHi-C loop-calling tools against a merged bulk-Hi-C/HiChIP reference list to rank methods by precision, recall, and F1 across subsampling levels.

## When NOT to use

- Input is already a scored metric table or publication-reported benchmark values rather than called loop bedpe sets to be re-evaluated
- The goal is calling loops on raw scHi-C contact files rather than evaluating previously called loop sets

## Inputs

- Called loop sets in bedpe format (summit coordinates) per loop-calling method per subsampling level, e.g., SnapHiC *.postprocessed.summits.bedpe
- Reference loop sources: HiCCUPS loops from bulk in situ Hi-C and MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP
- Subsampled scHi-C datasets from 742 mouse embryonic stem cells

## Outputs

- Per-method, per-subsampling precision/recall/F1 table (loop_calling_benchmark_f1.csv)
- F1-based ranking of loop-calling methods at each subsampling level

## How to apply

Collect each caller's loop output as bedpe summit coordinates, e.g., SnapHiC's *.postprocessed.summits.bedpe, for every subsampling level of the scHi-C data (drawn from 742 mouse embryonic stem cells). Build one combined reference list by merging HiCCUPS loops from bulk in situ Hi-C with MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP using Python. Match each method's called loops to the reference by genomic interval overlap of the loop anchor coordinates, then count true positives, false positives, and false negatives per method per subsampling level. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 as the harmonic mean of precision and recall; write per-method, per-subsampling values to loop_calling_benchmark_f1.csv and rank methods by F1.

## Related tools

- **SnapHiC** (Loop-calling method under benchmark evaluation; supplies *.postprocessed.summits.bedpe loop summits from subsampled mES scHi-C data) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Used to merge reference loop sources, perform genomic interval overlap matching of called loops, and compute precision/recall/F1 into loop_calling_benchmark_f1.csv)
- **cooler** (Converts SnapHiC count matrices into .cool files used in the pipeline whose loop outputs are benchmarked)

## Examples

```
python evaluate_loop_f1.py --called SnapHiC.postprocessed.summits.bedpe --reference combined_reference_loops.bedpe --match anchor_overlap --out loop_calling_benchmark_f1.csv
```

## Evaluation signals

- loop_calling_benchmark_f1.csv contains a precision, recall, and F1 value for every method at every subsampling level
- Precision = TP/(TP+FP) and recall = TP/(TP+FN) recomputed from the TP/FP/FN counts match the tabulated values
- F1 equals the harmonic mean of the tabulated precision and recall for each row
- SnapHiC's F1 is the maximum among HiCCUPS, FastHiC, FitHiC2, and HiC-ACT at every subsampling level
- Each matched loop pair satisfies genomic interval overlap of the loop anchor coordinates against the combined reference list

## Limitations

- Reference loops are a combination of bulk Hi-C (HiCCUPS) and MAPS interactions (H3K4me3 PLAC-seq, cohesin HiChIP, H3K27ac HiChIP), so results depend on the merged reference list definition
- SnapHiC requires mappability and blacklist filtering (10KB bins with mappability <=0.8 and ENCODE blacklist regions excluded), which constrains which loops can be called and evaluated
- The SnapHiC README provides no changelog, limiting verification of version differences affecting benchmark results

## Evidence

- [abstract] Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools: "Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools"
- [abstract] a method that can identify chromatin loops at high resolution and accuracy from scHi-C data: "a method that can identify chromatin loops at high resolution and accuracy from scHi-C data"
- [methods] postprocessing: clustering nearby loop candidates and identifying loop summits: "(5) postprocessing: clustering nearby loop candidates and identifying loop summits"
- [readme] Summits bedpe output file naming: "It is the prefix of the output files, including *.postprocessed.summits.bedpe"
