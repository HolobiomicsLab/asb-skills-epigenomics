---
name: density-clustering-of-loop-candidates
description: Use when a per-cell scHi-C loop-calling step has produced many candidate
  bin pairs and you need to consolidate them into non-redundant loop calls (summits)
  before downstream annotation.
when_to_use_negative:
- Input is bulk Hi-C data where high-resolution per-cell imputation (RWR) and per-cell
  paired t-testing are not applicable — SnapHiC targets single-cell Hi-C.
- You only need raw candidate bin pairs without consolidation — the pre-postprocess
  interaction output already contains all significant bin pairs.
- Genome is not mm10, hg19, or hg38 at 10-kb resolution with MboI-based mappability
  filters, since ext filter files and chromosome counts are only provided for those
  genomes.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3169
tools:
- name: SnapHiC
  role: Pipeline whose postprocess step performs density clustering of loop candidates
    and summit identification
  repo: github.com/HuMingLab/SnapHiC
- name: Python >= 3.6
  role: Runtime for snap.py including the postprocess/clustering step
- name: MPI (open-mpi)
  role: Parallel execution backend for SnapHiC steps on HPC
- name: Juicer
  role: Converts combined count matrix to .hic file for visualization in Juicebox
- name: cooler
  role: Converts combined count matrix to .cool file
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:7fa7e02b1b9bd66c3568b70328d6e09bdf2df86c23ff8560879393eebc0a7f5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Juicer
  - MPI (open-mpi)
  - Python
  - Python >= 3.6
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# density-clustering-of-loop-candidates

## Summary

Postprocessing step of SnapHiC that clusters nearby loop candidates and identifies loop summits, converting a noisy set of significant bin-pair calls into a compact, high-confidence set of chromatin loops.

## When to use

Use when a per-cell scHi-C loop-calling step has produced many candidate bin pairs and you need to consolidate them into non-redundant loop calls (summits) before downstream annotation.

## When NOT to use

- Input is bulk Hi-C data where high-resolution per-cell imputation (RWR) and per-cell paired t-testing are not applicable — SnapHiC targets single-cell Hi-C.
- You only need raw candidate bin pairs without consolidation — the pre-postprocess interaction output already contains all significant bin pairs.
- Genome is not mm10, hg19, or hg38 at 10-kb resolution with MboI-based mappability filters, since ext filter files and chromosome counts are only provided for those genomes.

## Inputs

- SnapHiC loop-candidate output (paired t-test statistics with global/local background comparisons, distance-matched fdr_dist, up to 2-Mb genomic distance)
- Binned 10-kb contact probability matrices across cells (from bin + rwr steps)
- Configured run_step2.sh variables (indir, outdir, genome=hg19/mm10/hg38, chrlen, filter_file, prefix)

## Outputs

- *.postprocessed.all_candidates.bedpe — clustered loop candidates after removing singletons
- *.postprocessed.summits.bedpe — loop summits (one per cluster), 17-column bedpe

## How to apply

After SnapHiC's loop-candidate testing (paired t-test on normalized contact probability across cells, global background >1.96 threshold, five local backgrounds: circle, donut, horizontal, vertical, lower_left, distance-matched fdr_dist up to 2 Mb), run the postprocess step ('python3 snap.py --steps interaction postprocess' or the postprocess step alone). The postprocess applies density clustering (Rodriguez-Liao density clustering) to merge nearby loop candidates, removing singletons, and then selects the loop summits within each cluster. Results are written to *.postprocessed.all_candidates.bedpe (clustered candidates) and *.postprocessed.summits.bedpe (summits) in 17-column bedpe format. Rationale: single significant bin pairs can be spurious; clustering nearby candidates suppresses singleton noise and yields one representative summit per loop.

## Related tools

- **SnapHiC** (Pipeline whose postprocess step performs density clustering of loop candidates and summit identification) — github.com/HuMingLab/SnapHiC
- **Python >= 3.6** (Runtime for snap.py including the postprocess/clustering step)
- **MPI (open-mpi)** (Parallel execution backend for SnapHiC steps on HPC)
- **Juicer** (Converts combined count matrix to .hic file for visualization in Juicebox)
- **cooler** (Converts combined count matrix to .cool file)

## Examples

```
python3 snap.py --steps "interaction postprocess"  # after configuring prefix=ODC, genome=hg19, filter_file=ext/hg19_filter_regions.txt in run_step2.sh
```

## Evaluation signals

- Output files *.postprocessed.summits.bedpe and *.postprocessed.all_candidates.bedpe exist and are valid 17-column bedpe.
- On the ODC_100 test dataset (100 oligodendrocytes, hg19), the summits file reproduces the reference 6,249 loops (odc_100.postprocessed.summits.bedpe), checkable via row count.
- No singleton candidates remain in the postprocessed candidate file (singletons are removed during postprocess).
- All reported summits fall within 2-Mb genomic distance, consistent with the fdr_dist loop-testing range.
- Summit coordinates correspond to the densest bins within their candidate clusters, consistent with the Rodriguez-Liao density clustering selection.

## Limitations

- Default RWR via full matrix inversion requires large memory and long runtime; the sliding_window approximation trades accuracy for feasibility, which propagates to candidate quality before clustering.
- SnapHiC currently accepts only mm10, hg19, and hg38 genomes, and loop calling assumes 10-kb bins with MboI-based mappability filters (bins with mappability <=0.8 and ENCODE blacklist overlaps removed).
- Large-scale parallel runs require an HPC environment with a job scheduler (PBS/SLURM) and MPI; single-processor runs are slow and memory-hungry.

## Evidence

- [methods] postprocess: clustering nearby loop candidates and identifying loop summits: "(5) postprocessing: clustering nearby loop candidates and identifying loop summits"
- [methods] five local background regions: "identifying loop candidates based on both global and local background models"
- [methods] removing singletons: "a file containing all the loop candidates (after removing singletons)"
- [methods] mappability and blacklist filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] output file names: "Name of the dataset. It is the prefix of the output files, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe"
