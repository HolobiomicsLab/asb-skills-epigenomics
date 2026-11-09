---
name: genomic-interval-overlap-matching
description: Use when comparing loop-calling results (bedpe anchor coordinates) against
  a curated reference interaction set and you need TP/FP/FN counts, precision, recall,
  and F1 per method and per subsampling level.
when_to_use_negative:
- Input is raw scHi-C contact files rather than called loop bedpe coordinates — run
  a loop caller (e.g. SnapHiC's bin/rwr/hic/interaction/postprocess steps) first
- The reference and called loops use different genome builds or bin resolutions —
  lift/match builds (mm10/hg19/hg38) before interval overlap or the anchors will never
  intersect
- You need statistical significance of loop enrichment rather than F1 against a reference
  list — this skill only does deterministic TP/FP/FN counting
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SnapHiC
  role: Loop caller whose *.postprocessed.summits.bedpe output supplies the called
    loop anchors that are matched against the combined reference list
  repo: https://github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Implements the interval-overlap matching of loop anchors, TP/FP/FN counting,
    precision/recall/F1 computation, and CSV export
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:dfd02db0fecdd7eb2ed26092f75e48b92f4c216b8a6bfff7456020040ee45d15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Python >= 3.6
  - SnapHiC
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# genomic-interval-overlap-matching

## Summary

A benchmark-matching procedure that scores called chromatin loops (bedpe summit coordinates) against a combined reference loop list by genomic interval overlap of loop anchors, yielding per-method precision, recall, and F1 — used here to evaluate SnapHiC against bulk Hi-C loop callers on subsampled mES scHi-C.

## When to use

Use when comparing loop-calling results (bedpe anchor coordinates) against a curated reference interaction set and you need TP/FP/FN counts, precision, recall, and F1 per method and per subsampling level.

## When NOT to use

- Input is raw scHi-C contact files rather than called loop bedpe coordinates — run a loop caller (e.g. SnapHiC's bin/rwr/hic/interaction/postprocess steps) first
- The reference and called loops use different genome builds or bin resolutions — lift/match builds (mm10/hg19/hg38) before interval overlap or the anchors will never intersect
- You need statistical significance of loop enrichment rather than F1 against a reference list — this skill only does deterministic TP/FP/FN counting

## Inputs

- Called loop sets in bedpe summit-coordinate format (e.g. *.postprocessed.summits.bedpe) per method per subsampling level
- HiCCUPS loops from bulk in situ Hi-C (reference)
- MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP (reference)
- Subsampled scHi-C datasets drawn from 742 mouse embryonic stem cells

## Outputs

- loop_calling_benchmark_f1.csv with per-method, per-subsampling precision, recall, and F1
- Method ranking by F1 (validation: SnapHiC F1 is maximum among all compared methods at every subsampling level)

## How to apply

Collect the called loop sets (bedpe summit coordinates, e.g. SnapHiC's *.postprocessed.summits.bedpe) for each method on each subsampled 742-cell mouse embryonic stem cell scHi-C dataset. Merge the reference sources — HiCCUPS loops from bulk in situ Hi-C plus MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP — into one combined reference list in Python. Match each method's called loops to the combined list by genomic interval overlap of the loop anchor coordinates, then count true positives, false positives, and false negatives per method per subsampling level. Compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 as the harmonic mean of precision and recall, writing per-method, per-subsampling values to loop_calling_benchmark_f1.csv and ranking methods by F1.

## Related tools

- **SnapHiC** (Loop caller whose *.postprocessed.summits.bedpe output supplies the called loop anchors that are matched against the combined reference list) — https://github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Implements the interval-overlap matching of loop anchors, TP/FP/FN counting, precision/recall/F1 computation, and CSV export)

## Examples

```
python match_loops.py --called SnapHiC.postprocessed.summits.bedpe --reference combined_reference_loops.bedpe --out loop_calling_benchmark_f1.csv
```

## Evaluation signals

- SnapHiC's tabulated F1 is the maximum among HiCCUPS, FastHiC, FitHiC2 and HiC-ACT at every subsampling level of the mES scHi-C data
- Per method per subsampling, TP + FN equals the size of the combined reference loop list (every reference loop is either matched or a false negative)
- precision = TP/(TP+FP) and recall = TP/(TP+FN) are in [0,1] and F1 equals the harmonic mean of the two
- loop_calling_benchmark_f1.csv contains one precision/recall/F1 row per method per subsampling level with no missing methods
- Matched anchors agree on chromosome and overlap within the interval-overlap rule; no TP is counted for anchors on different chromosomes

## Limitations

- F1 depends entirely on the composition of the combined reference list (HiCCUPS bulk Hi-C + MAPS PLAC-seq/HiChIP interactions); a reference built from different data sources would change the ranking
- Interval-overlap matching treats a called loop as a TP on anchor overlap alone and does not verify loop identity beyond coordinate agreement
- Results are specific to the 742-cell mES scHi-C benchmark; generalization to other cell types or organisms is not established by this workflow
- SnapHiC's own mappability filtering (10KB bins with mappability <=0.8 and ENCODE blacklist overlaps removed) restricts the loop-candidate space before matching

## Evidence

- [other] Match each method's called loops to the combined reference list by genomic interval overlap of the loop anchor coordinates in Python.: [evidence span withheld — anchor status: not_found]
- [other] compute precision = TP/(TP+FP), recall = TP/(TP+FN), and F1 as the harmonic mean: [evidence span withheld — anchor status: not_found]
- [other] Merge the reference loop sources — HiCCUPS loops from bulk in situ Hi-C and MAPS interactions from H3K4me3 PLAC-seq, cohesin HiChIP, and H3K27ac HiChIP: [evidence span withheld — anchor status: not_found]
- [readme] It is the prefix of the output files, including *.postprocessed.summits.bedpe: "It is the prefix of the output files, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe, and *allChr.[hic/cool]*."
- [readme] We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions.: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
