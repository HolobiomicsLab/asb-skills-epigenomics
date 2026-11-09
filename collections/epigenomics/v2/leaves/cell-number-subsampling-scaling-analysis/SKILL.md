---
name: cell-number-subsampling-scaling-analysis
description: Use when evaluating how loop-calling performance scales with the number
  of single cells, e.g. comparing SnapHiC on per-cell subsamples of mES scHi-C data
  against HiCCUPS on pooled contacts.
when_to_use_negative:
- Data is bulk Hi-C from a large cell population rather than sparse per-cell scHi-C
  contacts, where pooling-based HiCCUPS is adequate
- The genome of interest is not mm10, hg19, or hg38 at 10KB resolution with MboI,
  since precomputed filter-region files are only provided for those combinations
- The question is loop accuracy/validation against ChIA-PET etc. rather than scaling
  of loop counts with cell number
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3912
tools:
- name: SnapHiC
  role: Five-step loop-calling pipeline (bin, rwr, hic, interaction, postprocess)
    run independently on each per-cell subsample
  repo: https://github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Runs HiCCUPS with default and optimal parameters on the pooled contacts of
    each subsample as the bulk-method comparator
- name: cooler
  role: Converts the combined count matrix into .cool files as part of SnapHiC step
    3
- name: Singularity
  role: Container image of SnapHiC for reproducible execution
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:9ce4ec29d53cf111f18388b401a9d68d25b32b99515cf63282de95889fcb2214
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Java >= 8
  - Juicer
  - MPI (open-mpi)
  - Python >= 3.6
  - Singularity
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3912
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cell-number-subsampling-scaling-analysis

## Summary

Benchmark a single-cell Hi-C loop caller by running it independently on a series of subsampled cell numbers (e.g. the 10-742 mES cell series) and comparing loop counts against a pooled-data bulk method (Juicer HiCCUPS with default and optimal parameters) to quantify how few cells are needed for reliable loop detection.

## When to use

Use when evaluating how loop-calling performance scales with the number of single cells, e.g. comparing SnapHiC on per-cell subsamples of mES scHi-C data against HiCCUPS on pooled contacts.

## When NOT to use

- Data is bulk Hi-C from a large cell population rather than sparse per-cell scHi-C contacts, where pooling-based HiCCUPS is adequate
- The genome of interest is not mm10, hg19, or hg38 at 10KB resolution with MboI, since precomputed filter-region files are only provided for those combinations
- The question is loop accuracy/validation against ChIA-PET etc. rather than scaling of loop counts with cell number

## Inputs

- Per-cell tab-separated (optionally gzipped) contact files (chr1, pos1, chr2, pos2) for each subsample size in the 10-742 mES cell series
- mm10 chrom.sizes file
- mm10 10KB MboI filter-regions bed file (bins with mappability <=0.8 plus ENCODE blacklist regions)
- Pooled contact files per subsample for Juicer HiCCUPS

## Outputs

- *.postprocessed.summits.bedpe loop calls per subsample size from SnapHiC
- HiCCUPS loop calls (default and optimal parameters) on pooled data per subsample size
- loop_counts_subsample_benchmark.csv compiling loop counts per tool and cell number

## How to apply

Assemble per-cell tab-separated (optionally gzipped) contact files from the mES scHi-C subsampling series (10-742 cells), together with the mm10 chrom.sizes file and the 10KB MboI filter-regions bed (bins with mappability <=0.8 plus ENCODE blacklist regions). For each subsample size, run the SnapHiC pipeline independently via `python snap.py` through all five steps -- 'bin rwr hic interaction postprocess' -- with 10KB binsize and a 2Mb maximal genomic distance, allocating ~30-40GB per processor for bin/rwr and ~20GB for hic/interaction/postprocess, with no more than ~5 processors per node for step 3. In parallel, pool the contacts for each subsample and run Juicer HiCCUPS twice: once with default parameters and once with the optimal parameter set. Count loops per tool and subsample size from the *.postprocessed.summits.bedpe files (using fdr_dist as the loop-strength measure) and the HiCCUPS outputs, and compile them into loop_counts_subsample_benchmark.csv.

## Related tools

- **SnapHiC** (Five-step loop-calling pipeline (bin, rwr, hic, interaction, postprocess) run independently on each per-cell subsample) — https://github.com/HuMingLab/SnapHiC
- **Juicer** (Runs HiCCUPS with default and optimal parameters on the pooled contacts of each subsample as the bulk-method comparator)
- **cooler** (Converts the combined count matrix into .cool files as part of SnapHiC step 3)
- **Singularity** (Container image of SnapHiC for reproducible execution)

## Examples

```
python snap.py --steps "bin rwr hic interaction postprocess" --indir subsample_75cells/ --suffix contacts.txt.gz --outdir out_75/ --chrlen ext/mm10.chrom.sizes --genome mm10 --filter_file ext/mm10_filter_regions.txt --prefix mES75 --binsize 10KB
```

## Evaluation signals

- At 75 mES cells, SnapHiC yields 1,050-1,420 loops at 10-kb resolution, falling within the reported benchmark range
- At 75 cells, HiCCUPS on the pooled data yields only 0-2 loops with default parameters and 3-10 loops with optimal parameters
- SnapHiC loop counts are derived from *.postprocessed.summits.bedpe files with fdr_dist as the loop-strength measure
- Reproduced counts across the 10-742 cell subsampling series fall within the reported benchmark ranges and are compiled in loop_counts_subsample_benchmark.csv

## Limitations

- Random walk computation with the default inverse-matrix method requires very large memory (~30-40GB per processor for bin/rwr), so an HPC environment with job scheduling is strongly recommended
- SnapHiC currently accepts only mm10, hg19, and hg38 reference genomes
- No more than ~5 processors per node should be used for step 3; discussion-section caveats were not extractable from the provided text

## Evidence

- [other] From only 75 mES cells, SnapHiC identified 1,050-1,420 loops at 10-kb resolution, whereas HiCCUPS on the pooled data found only 0-2 loops with default parameters and 3-10 loops with optimal parameters.: [evidence span withheld — anchor status: not_found]
- [methods] Five steps of SnapHiC: "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR), (3) combining all cells to generate .hic file"
- [methods] Filter regions definition: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] SnapHiC step CLI options: "use the argument `--step` followed by any of the five options: 'bin', 'rwr', 'hic', 'interaction', and 'postprocess'."
