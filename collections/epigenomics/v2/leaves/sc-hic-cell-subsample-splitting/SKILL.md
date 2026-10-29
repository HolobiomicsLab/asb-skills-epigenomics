---
name: sc-hic-cell-subsample-splitting
description: Use when you have per-cell scHi-C contact files (e.g. 742 mES cells)
  and need to measure how reproducibly two loop-calling methods or parameter settings
  recover loops from equally sized cell subsamples.
when_to_use_negative:
- You need to call loops on the full cell population rather than benchmark subsample
  reproducibility — run the pipeline once on all cells instead
- Input is bulk Hi-C or already-aggregated contact matrices with no per-cell files
  to split
- Comparing reproducibility across different cell types or conditions rather than
  random splits of the same population — use differential loop analysis instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SnapHiC
  role: Loop-calling pipeline run independently on each 371-cell half via snap.py
    steps 'bin rwr' and 'hic interaction postprocess'; loop summits extracted from
    *.postprocessed.summits.bedpe
  repo: https://github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Generates .hic count matrices from each half for HiCCUPS loop calling with
    default and optimal parameters
- name: cooler
  role: Converts combined count matrices into .cool files as part of the SnapHiC output
    step
- name: Dip-C
  role: Alternative single-cell Hi-C preprocessing pipeline that can generate the
    per-cell contact files used as split input
  repo: https://github.com/tanlongzhi/dip-c
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:cec7706f07b1fe9e572de09ab9baa58a6966383451710ac65d9e8cd3815470d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Dip-C
  - Java
  - Juicer
  - MPI (open-mpi)
  - Python
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# sc-hic-cell-subsample-splitting

## Summary

Randomly partition a pool of single-cell Hi-C contact files into two equal, disjoint replicate halves to benchmark loop-calling reproducibility (symmetric overlap) between methods such as SnapHiC and Juicer HiCCUPS on identical cell subsets.

## When to use

Use when you have per-cell scHi-C contact files (e.g. 742 mES cells) and need to measure how reproducibly two loop-calling methods or parameter settings recover loops from equally sized cell subsamples.

## When NOT to use

- You need to call loops on the full cell population rather than benchmark subsample reproducibility — run the pipeline once on all cells instead
- Input is bulk Hi-C or already-aggregated contact matrices with no per-cell files to split
- Comparing reproducibility across different cell types or conditions rather than random splits of the same population — use differential loop analysis instead

## Inputs

- Per-cell scHi-C contact files (tab-separated or tab-separated+gzipped; 4 columns: chr1, chr2, pos1, pos2), e.g. 742 mES cell contact files
- chrom.sizes file for the reference genome (e.g. mm10.chrom.sizes)
- Binned filter-regions bed file (10KB bins with mappability <=0.8 and ENCODE blacklist overlaps, e.g. mm10_filter_regions.txt)

## Outputs

- Two disjoint subsample contact-file directories (e.g. 371 cells each)
- Per-half loop summit call sets (*.postprocessed.summits.bedpe) for each method/parameter setting
- Reproducibility summary table (e.g. reproducibility_summary.csv) with symmetric overlap fractions per method

## How to apply

Randomly partition the per-cell contact files (tab-separated, 2 chromosome columns + 2 mapped-position columns per contact) into two disjoint groups — e.g. split 742 mouse embryonic stem cells into two replicate datasets of 371 cells each. Run the loop-calling pipeline independently on each half: for SnapHiC, execute snap.py 'bin rwr' followed by 'hic interaction postprocess' using 10KB bins with mm10 chrom.sizes and the mm10 filter-regions file (bins with mappability <=0.8 and ENCODE blacklist overlaps removed), then extract loop summits from *.postprocessed.summits.bedpe using fdr_dist as the loop-strength measure. For the HiCCUPS comparison, generate .hic count matrices via Juicer (requires Java >= 8) from each half and run HiCCUPS with both default and optimal parameters. For each method/parameter combination, compute the symmetric overlap — the fraction of loops called in half A recovered in half B and vice versa — and compare against published benchmarks (SnapHiC 50.8%, HiCCUPS default 38.7%, HiCCUPS optimal 39.7%).

## Related tools

- **SnapHiC** (Loop-calling pipeline run independently on each 371-cell half via snap.py steps 'bin rwr' and 'hic interaction postprocess'; loop summits extracted from *.postprocessed.summits.bedpe) — https://github.com/HuMingLab/SnapHiC
- **Juicer** (Generates .hic count matrices from each half for HiCCUPS loop calling with default and optimal parameters)
- **cooler** (Converts combined count matrices into .cool files as part of the SnapHiC output step)
- **Dip-C** (Alternative single-cell Hi-C preprocessing pipeline that can generate the per-cell contact files used as split input) — https://github.com/tanlongzhi/dip-c

## Examples

```
python snap.py --step bin --indir replicates/halfA/contacts --suffix contacts.txt.gz --outdir out_halfA --chrs "2 4" --pos "3 5" --chrlen ext/mm10.chrom.sizes --genome mm10 --filter_file ext/mm10_filter_regions.txt --prefix halfA
```

## Evaluation signals

- Symmetric overlap for SnapHiC on the two 371-cell halves should reproduce approximately 50.8%
- HiCCUPS with default parameters should yield ~38.7% symmetric overlap and HiCCUPS with optimal parameters ~39.7% on the same splits
- The two replicate groups must be disjoint and equal-sized (e.g. 371 + 371 = 742 cells, no cell appearing in both halves)
- Loop summits are extracted consistently across methods using the same strength measure (fdr_dist) before overlap computation
- Contact files pass format checks: tab-separated, 4 columns, correct chromosome/position column indices (chrs='2 4', pos='3 5') supplied to the pipeline

## Limitations

- Reproducibility from 371-cell subsamples is lower than from the full 742-cell set; values are subsample-specific and may shift with different split seeds or cell numbers
- The filter-regions file is resolution- and restriction-enzyme-specific (10KB bins, MboI; mappability <=0.8 plus ENCODE blacklist); other resolutions or enzymes require regenerating mappability files
- SnapHiC default (inverse matrix) RWR computation is memory-intensive; sliding-window approximation is available but changes the computation path
- SnapHiC currently accepts only mm10, hg19 and hg38 reference genomes

## Evidence

- [results] loop reproducibility was 50.8% for SnapHiC versus 38.7% for HiCCUPS with default parameters and 39.7% for HiCCUPS with optimal parameters: [evidence span withheld — anchor status: not_found]
