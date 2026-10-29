---
name: genomic-distance-normalization-of-contact-probabilities
description: Use when calling chromatin loop candidates from single-cell Hi-C contact
  matrices at 10-kb bin resolution and you need to control for the monotonic distance
  decay of contact probability before declaring a bin pair significant.
when_to_use_negative:
- Input is bulk Hi-C data rather than single-cell/per-cell contact files, since the
  paired t-test operates across cells' normalized contact probability matrices
- Bin pairs are separated by more than 2 Mb, beyond the distance-matched FDR (fdr_dist)
  calibration range
- You only have raw fastq reads with no aligned per-cell contacts — run a scHi-C preprocessing
  pipeline (e.g. Dip-C) first to produce contact files
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SnapHiC
  role: Pipeline implementing the loop-candidate testing (interaction step with global/local
    background models and paired t-test) and postprocess clustering
  repo: https://github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Converts the combined count matrix into .hic files for visualization in Juicebox
- name: cooler
  role: Converts the combined count matrix into .cool files
- name: dip-c
  role: Alternative single-cell Hi-C preprocessing pipeline that can generate the
    per-cell contact files used as input
  repo: https://github.com/tanlongzhi/dip-c
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:2e4d0027b6badb30e02c57250622cbfef42018e48ff5698999fff103bf8fb97e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Java
  - Juicer
  - MPI (open-mpi)
  - Python
  - SnapHiC
  - cooler
  - dip-c
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Genomic-distance-normalization of contact probabilities

## Summary

In SnapHiC, loop candidates are identified by testing whether a bin pair's normalized contact probability is significantly higher than expected against both a global background model and five distance-matched local background regions (circle, donut, horizontal, vertical, lower_left) across all cells, using paired t-tests with distance-matched FDR (fdr_dist) for bin pairs up to 2 Mb apart.

## When to use

Use when calling chromatin loop candidates from single-cell Hi-C contact matrices at 10-kb bin resolution and you need to control for the monotonic distance decay of contact probability before declaring a bin pair significant.

## When NOT to use

- Input is bulk Hi-C data rather than single-cell/per-cell contact files, since the paired t-test operates across cells' normalized contact probability matrices
- Bin pairs are separated by more than 2 Mb, beyond the distance-matched FDR (fdr_dist) calibration range
- You only have raw fastq reads with no aligned per-cell contacts — run a scHi-C preprocessing pipeline (e.g. Dip-C) first to produce contact files

## Inputs

- Per-cell tab-separated (optionally gzipped) contact files with 2 columns for chromosome name and 2 columns for mapped positions (bp)
- chrom.sizes file (mm10, hg19, or hg38; SnapHiC accepts these genomes)
- Binned 10-kb filter_regions bed file (bins with mappability <=0.8 and ENCODE blacklist overlaps, e.g. ext/hg19_filter_regions.txt for MboI)
- RWR-imputed per-cell normalized contact probability matrices from prior 'bin' and 'rwr' steps

## Outputs

- *.postprocessed.all_candidates.bedpe (17-column BEDPE of all loop candidates after singleton removal)
- *.postprocessed.summits.bedpe (loop summits after Rodriguez-Liao density clustering)
- allChr.hic (via Juicer) and allChr.cool (via cooler) count matrix files

## How to apply

After binning contacts into 10-kb bins and imputing per-cell contact probabilities via random walk with restart (RWR), aggregate all cells into a count matrix and test each bin pair within 2 Mb. Compare each cell's normalized contact probability against the global background (>1.96 threshold) and against five local background regions (circle, donut, horizontal, vertical, lower_left), and apply a paired t-test to the matrices of normalized contact probability across all cells to flag loop candidates with higher-than-expected contact probability. Compute distance-matched FDR (fdr_dist) so significance is calibrated within genomic-distance bins rather than globally. Remove singleton candidates, then postprocess by clustering nearby loop candidates (Rodriguez-Liao density clustering) and selecting loop summits, writing *.postprocessed.all_candidates.bedpe and *.postprocessed.summits.bedpe.

## Related tools

- **SnapHiC** (Pipeline implementing the loop-candidate testing (interaction step with global/local background models and paired t-test) and postprocess clustering) — https://github.com/HuMingLab/SnapHiC
- **Juicer** (Converts the combined count matrix into .hic files for visualization in Juicebox)
- **cooler** (Converts the combined count matrix into .cool files)
- **dip-c** (Alternative single-cell Hi-C preprocessing pipeline that can generate the per-cell contact files used as input) — https://github.com/tanlongzhi/dip-c

## Examples

```
python3 snap.py --steps "hic interaction postprocess"
```

## Evaluation signals

- Row-count comparison of *.postprocessed.summits.bedpe against the reference output of 6,249 loops for the ODC_100 test dataset (odc_100.postprocessed.summits.bedpe)
- Output BEDPE files conform to the expected 17-column format
- All reported loop candidates lie within the 2-Mb genomic-distance limit and survive the >1.96 global-background threshold and distance-matched FDR (fdr_dist)
- No candidate bin overlaps bins removed by the mappability <=0.8 or ENCODE blacklist filter file
- *.postprocessed.all_candidates.bedpe and *.postprocessed.summits.bedpe files are produced with the configured dataset prefix (e.g. ODC)

## Limitations

- Loop calling is restricted to autosomal chromosomes and to distances up to 2 Mb; bins with mappability <=0.8 or ENCODE blacklist overlaps are excluded from the outset
- The default RWR implementation inverts a chromosome-sized square matrix, which requires large memory and can be slow; the sliding_window method is a lower-memory alternative
- No changelog found in the source repository and no SnapHiC version is given in the methods beyond 0.2.0 (October 29, 2021), limiting reproducibility checks

## Evidence

- [methods] identifying loop candidates based on both global and local background models: "(4) identifying loop candidates based on both global and local background models, and (5) postprocessing: clustering nearby loop candidates and identifying loop summits"
- [results] paired t-test across cells: [evidence span withheld — anchor status: not_found]
- [readme] mappability and blacklist filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] filter file definition: "A binned bed file of the genomic regions that are excluded from loop calling."
