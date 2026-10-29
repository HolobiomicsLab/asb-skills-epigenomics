---
name: fdr-based-loop-strength-ranking
description: Use when SnapHiC loop calls in *.postprocessed.summits.bedpe files must
  be counted or ranked with fdr_dist as the loop-strength measure — e.g. the 10–742-cell
  mES subsampling loop-count benchmark or symmetric reproducibility between split
  371-cell replicate halves.
when_to_use_negative:
- The loops to rank come from Juicer HiCCUPS or bulk pooled Hi-C rather than SnapHiC's
  *.postprocessed.summits.bedpe — fdr_dist is not a HiCCUPS score.
- The SnapHiC run has not reached the 'postprocess' step (singletons not removed,
  summits not identified) — *.postprocessed.all_candidates.bedpe is an unranked candidate
  list that would inflate loop counts.
- The genome is not one of mm10, hg19, or hg38, since SnapHiC only accepts those reference
  genomes and the provided 10KB MboI filter files cover those builds.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SnapHiC
  role: Pipeline (snap.py) whose 'postprocess' step clusters nearby loop candidates,
    removes singletons, and writes the *.postprocessed.summits.bedpe file whose fdr_dist
    column ranks loop strength
  repo: https://github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Runs HiCCUPS with default and optimal parameters on pooled .hic files to produce
    the comparison loop call sets counted against the fdr_dist-ranked SnapHiC summits
- name: Python >= 3.6
  role: Runtime for snap.py executed in parallel/MPI mode to generate the summits
    BEDPE whose fdr_dist values are ranked
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:912a979611ecf3ce3e9e48a816903883bb3967cd955fd4c66157001c80e63920
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Java
  - Java >= 8
  - Juicer
  - MPI (open-mpi)
  - Python
  - Python >= 3.6
  - Singularity
  - SnapHiC
  - cooler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# fdr-based-loop-strength-ranking

## Summary

Extracts loop summits from SnapHiC's *.postprocessed.summits.bedpe output and uses the fdr_dist column as the loop-strength measure for ranking, counting, and cross-replicate overlap analyses of scHi-C loops. This is the ranking convention behind the SnapHiC-vs-HiCCUPS loop-count and 371-cell reproducibility benchmarks.

## When to use

Use when SnapHiC loop calls in *.postprocessed.summits.bedpe files must be counted or ranked with fdr_dist as the loop-strength measure — e.g. the 10–742-cell mES subsampling loop-count benchmark or symmetric reproducibility between split 371-cell replicate halves.

## When NOT to use

- The loops to rank come from Juicer HiCCUPS or bulk pooled Hi-C rather than SnapHiC's *.postprocessed.summits.bedpe — fdr_dist is not a HiCCUPS score.
- The SnapHiC run has not reached the 'postprocess' step (singletons not removed, summits not identified) — *.postprocessed.all_candidates.bedpe is an unranked candidate list that would inflate loop counts.
- The genome is not one of mm10, hg19, or hg38, since SnapHiC only accepts those reference genomes and the provided 10KB MboI filter files cover those builds.

## Inputs

- *.postprocessed.summits.bedpe loop-summit files from SnapHiC postprocess (BEDPE, one row per clustered loop with fdr_dist strength value), one per subsample or replicate half
- Per-cell mES contact files (tab-separated, optionally gzipped) from the 742-cell population and its 10–742-cell subsampling series
- ext/mm10.chrom.sizes and ext/mm10_filter_regions.txt (10KB MboI filter: bins with mappability <=0.8 and ENCODE blacklist regions)

## Outputs

- fdr_dist-ranked loop summit call sets (one per SnapHiC run / subsample / replicate half)
- loop_counts_subsample_benchmark.csv with loop counts per tool (SnapHiC, HiCCUPS default, HiCCUPS optimal) and subsample size
- reproducibility_summary.csv with symmetric overlap values for SnapHiC, HiCCUPS default, and HiCCUPS optimal

## How to apply

Run snap.py through all five steps ('bin rwr hic interaction postprocess') under MPI with 10KB bins, a 2Mb maximal genomic distance, the mm10 chrom.sizes file, and ext/mm10_filter_regions.txt, which excludes bins with mappability <=0.8 and ENCODE blacklist overlaps. Only after postprocess — which clusters nearby loop candidates and removes singletons — take loop summits from *.postprocessed.summits.bedpe, not the pre-clustering *.postprocessed.all_candidates.bedpe, and read the fdr_dist column as the measure of loop strength. Count the fdr_dist-ranked summits per subsample size (at 75 mES cells expect 1,050–1,420 loops at 10-kb resolution) and, for reproducibility, compute the symmetric overlap (fraction of loops in half A recovered in half B and vice versa) between the two 371-cell halves of the 742 mES cells. Compile the counts into loop_counts_subsample_benchmark.csv and the overlaps into reproducibility_summary.csv, then check them against the published benchmarks (SnapHiC 50.8% reproducibility vs 38.7% HiCCUPS default and 39.7% HiCCUPS optimal) to confirm the ranking convention matches the paper's.

## Related tools

- **SnapHiC** (Pipeline (snap.py) whose 'postprocess' step clusters nearby loop candidates, removes singletons, and writes the *.postprocessed.summits.bedpe file whose fdr_dist column ranks loop strength) — https://github.com/HuMingLab/SnapHiC
- **Juicer** (Runs HiCCUPS with default and optimal parameters on pooled .hic files to produce the comparison loop call sets counted against the fdr_dist-ranked SnapHiC summits)
- **Python >= 3.6** (Runtime for snap.py executed in parallel/MPI mode to generate the summits BEDPE whose fdr_dist values are ranked)

## Examples

```
python snap.py --steps "bin rwr hic interaction postprocess"
```

## Evaluation signals

- Only *.postprocessed.summits.bedpe rows are ranked/counted — rows from *.postprocessed.all_candidates.bedpe (singletons not removed) must not inflate loop counts.
- At 75 mES cells the fdr_dist-based SnapHiC loop count falls in the 1,050–1,420 range at 10-kb resolution, versus 0–2 (HiCCUPS default) and 3–10 (HiCCUPS optimal).
- Symmetric overlap between the two 371-cell halves' fdr_dist-ranked call sets reproduces ~50.8% for SnapHiC versus 38.7% (HiCCUPS default) and 39.7% (HiCCUPS optimal).
- loop_counts_subsample_benchmark.csv and reproducibility_summary.csv fall within the published benchmark ranges across the 10–742 cell subsampling series.
- No ranked summit falls in a bin removed by ext/mm10_filter_regions.txt (mappability <=0.8 or ENCODE blacklist) at 10KB resolution.

## Limitations

- fdr_dist is a SnapHiC-specific strength measure; it cannot rank or calibrate HiCCUPS calls, so cross-method comparisons rest on raw loop counts and symmetric overlaps rather than a shared score.
- Ranking is only defined for bins surviving the MboI 10KB filter (mappability <=0.8 and ENCODE blacklist bins removed); provided filter files cover mm10, hg19, and hg38 at 10KB only.
- The 50.8% / 38.7% / 39.7% reproducibility values come from one random 50/50 split of the 742 mES cells, and the tested SnapHiC release is version 0.2.0 (October 29th, 2021) with no changelog available.

## Evidence

- [methods] Loop summits are produced by the postprocess step that clusters nearby loop candidates: "postprocessing: clustering nearby loop candidates and identifying loop summits"
- [methods] Filtered regions excluded from loop calling are 10KB bins with mappability <=0.8 and ENCODE blacklist overlaps: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] The dataset prefix names the summits BEDPE output file used for ranking: "It is the prefix of the output files, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe"
- [other] fdr_dist serves as the loop-strength measure when counting called loops: [evidence span withheld — anchor status: not_found]
- [other] Reproducibility benchmark computed from fdr_dist-ranked loop call sets: [evidence span withheld — anchor status: not_found]
