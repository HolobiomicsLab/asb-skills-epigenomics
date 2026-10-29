---
name: chromatin-loop-calling-from-schic
description: Use when you have per-cell (tab-separated, optionally gzipped) scHi-C
  contact files for mm10/hg19/hg38 and need cell-type-specific chromatin loop candidates
  and summits.
when_to_use_negative:
- Input is bulk Hi-C or already-aggregated contact matrices rather than per-cell contact
  files — SnapHiC's per-cell paired t-test requires single-cell contacts
- Genome assembly other than mm10, hg19, or hg38, since SnapHiC uses these to determine
  autosomal chromosome count and generate .hic files
- Goal is 3D haplotype-resolved genome reconstruction rather than loop identification
  (use Dip-C/hickit instead)
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: SnapHiC
  role: 'Main pipeline: binning, RWR imputation, global/local background loop-candidate
    testing, clustering, summit selection'
  repo: github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Converts the combined cell count matrix into .hic files for Juicebox visualization
- name: cooler
  role: Converts the combined cell count matrix into .cool format
- name: Dip-C
  role: Alternative single-cell Hi-C preprocessing pipeline that can generate the
    per-cell contact files SnapHiC consumes
  repo: github.com/tanlongzhi/dip-c
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:368d856b384ba1cf15ad11e7891f6c331990f4dae0a7537fd5006018c0eaec65
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dip-C
  - Java
  - Juicer
  - MPI (open-mpi)
  - Python
  - SnapHiC
  - cooler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# chromatin-loop-calling-from-schic

## Summary

SnapHiC-style loop calling from single-cell Hi-C: bin contacts at 10-kb, impute per-cell contact probabilities via random walk with restart (RWR), then test each bin pair against global and local background models to identify high-resolution chromatin loops.

## When to use

Use when you have per-cell (tab-separated, optionally gzipped) scHi-C contact files for mm10/hg19/hg38 and need cell-type-specific chromatin loop candidates and summits.

## When NOT to use

- Input is bulk Hi-C or already-aggregated contact matrices rather than per-cell contact files — SnapHiC's per-cell paired t-test requires single-cell contacts
- Genome assembly other than mm10, hg19, or hg38, since SnapHiC uses these to determine autosomal chromosome count and generate .hic files
- Goal is 3D haplotype-resolved genome reconstruction rather than loop identification (use Dip-C/hickit instead)

## Inputs

- Per-cell contact files (tab-separated or tab-separated gzipped; one line per contact with chromosome and mapped-position columns)
- chrom.sizes file for mm10, hg19, or hg38
- Binned bed filter file of excluded regions (bins with mappability <=0.8 plus ENCODE blacklist regions, e.g. ext/mm10_filter_regions.txt)

## Outputs

- *.postprocessed.all_candidates.bedpe (all loop candidates after singleton removal)
- *.postprocessed.summits.bedpe (clustered loop summits)
- allChr.hic (Juicer visualization file)
- allChr.cool (cooler file)

## How to apply

Run Part 1 with python3 snap.py --steps "bin rwr" to bin contacts into 10-kb bins (removing bins with mappability <=0.8 and ENCODE blacklist overlaps) and impute contact probabilities per chromosome per cell via RWR (inverse-matrix default, or sliding_window for lower memory). Run Part 2 with --steps "hic interaction postprocess" to combine all cells into a count matrix, convert it to .hic (Juicer) and .cool (cooler) files, and call loop candidates by comparing each cell's normalized contact probability against the global background (>1.96) and five local background regions (circle, donut, horizontal, vertical, lower_left). A paired t-test is applied to the matrices of normalized contact probability across cells; singletons are removed and distance-matched FDR (fdr_dist) is computed at up to 2-Mb genomic distance. Postprocess with Rodriguez-Liao density clustering of nearby candidates to select loop summits, configuring run_step1.sh/run_step2.sh variables (indir, outdir, genome, chrlen, filter_file, prefix).

## Related tools

- **SnapHiC** (Main pipeline: binning, RWR imputation, global/local background loop-candidate testing, clustering, summit selection) — github.com/HuMingLab/SnapHiC
- **Juicer** (Converts the combined cell count matrix into .hic files for Juicebox visualization)
- **cooler** (Converts the combined cell count matrix into .cool format)
- **Dip-C** (Alternative single-cell Hi-C preprocessing pipeline that can generate the per-cell contact files SnapHiC consumes) — github.com/tanlongzhi/dip-c

## Examples

```
python3 snap.py --steps "bin rwr"
```

## Evaluation signals

- On the ODC_100 test dataset (100 oligodendrocyte contacts, hg19), odc_100.postprocessed.summits.bedpe should contain 6,249 loops (row-count comparison against the published test output)
- Output bedpe files follow the 17-column bedpe format
- All reported loop candidates are within 2-Mb genomic distance (fdr_dist cutoff)
- All bins overlapping filter_file regions (mappability <=0.8, ENCODE blacklist) are absent from loop calls
- Companion allChr.hic and allChr.cool files open in Juicebox and cooler without errors

## Limitations

- Default RWR solves a matrix inverse the size of the binned chromosome, requiring large memory; sliding_window approximation trades accuracy for memory
- Only mm10, hg19, and hg38 are accepted as reference genomes
- SnapHiC is designed for single-cell Hi-C data; contact files must be per-cell with known chromosome/position column numbers (chrs and pos variables)

## Evidence

- [methods] five-step RWR pipeline: "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR)"
- [methods] global/local background testing: "identifying loop candidates based on both global and local background models, and (5) postprocessing: clustering nearby loop candidates and identifying loop summits"
- [readme] mappability/blacklist filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] input contact file format: "In each file, one line represents one contact with 2 columns for chromosome name and 2 columns for the mapped positions (bp)."
- [results] Paired t-test on normalized contacts: [evidence span withheld — anchor status: not_found]
