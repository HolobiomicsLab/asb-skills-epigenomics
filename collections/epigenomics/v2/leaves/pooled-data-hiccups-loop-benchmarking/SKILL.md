---
name: pooled-data-hiccups-loop-benchmarking
description: 'Use when quantifying how chromatin loop recovery scales with scHi-C
  cell number: per-cell contact files exist at multiple subsample sizes and pooled
  HiCCUPS loop calls must be compared against SnapHiC.'
when_to_use_negative:
- Input contacts are bulk (population) Hi-C rather than per-cell scHi-C files — there
  is no subsampling series to pool, so HiCCUPS should be run directly instead.
- The goal is a one-off loop call on a single deeply sequenced dataset rather than
  measuring how loop recovery scales with cell number.
- The assembly is not mm10, hg19 or hg38 — SnapHiC 'currently accepts mm10, hg19 and
  hg38', so no matching 10-kb MboI filter-regions file exists.
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_0102
tools:
- name: SnapHiC
  role: Runs the five-step loop-calling pipeline (bin, rwr, hic, interaction, postprocess)
    on each subsample's per-cell contact files to produce the *.postprocessed.summits.bedpe
    loop calls being benchmarked
  repo: https://github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Supplies the HiCCUPS loop caller that is run on each subsample's pooled contacts
    with both default and optimal parameter sets as the comparator
- name: cooler
  role: Converts the SnapHiC combined count matrix into a .cool file during the 'hic'
    step for each subsample
- name: Dip-C
  role: Alternative single-cell Hi-C preprocessing pipeline that can generate the
    per-cell tab-separated contact files consumed by the benchmark
  repo: https://github.com/tanlongzhi/dip-c
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:cd7f6d2a4a4c84a056778e2a4d6b97150df536ef359f2ea9bc301f2f953678d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Dip-C
  - Java >= 8
  - Juicer
  - MPI (open-mpi)
  - Python >= 3.6
  - Singularity
  - SnapHiC
  - cooler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pooled-data-hiccups-loop-benchmarking

## Summary

Benchmarks chromatin-loop detection sensitivity by running SnapHiC on each mES scHi-C subsample (10–742 cells) and Juicer HiCCUPS (default and optimal parameters) on each subsample's pooled contacts, then comparing loop counts at 10-kb resolution — e.g., at 75 cells SnapHiC finds 1,050–1,420 loops versus 0–2 (default) and 3–10 (optimal) for pooled HiCCUPS.

## When to use

Use when quantifying how chromatin loop recovery scales with scHi-C cell number: per-cell contact files exist at multiple subsample sizes and pooled HiCCUPS loop calls must be compared against SnapHiC.

## When NOT to use

- Input contacts are bulk (population) Hi-C rather than per-cell scHi-C files — there is no subsampling series to pool, so HiCCUPS should be run directly instead.
- The goal is a one-off loop call on a single deeply sequenced dataset rather than measuring how loop recovery scales with cell number.
- The assembly is not mm10, hg19 or hg38 — SnapHiC 'currently accepts mm10, hg19 and hg38', so no matching 10-kb MboI filter-regions file exists.

## Inputs

- Per-cell tab-separated (optionally gzipped) contact files for each mES scHi-C subsample (one line per contact: 2 chromosome-name columns and 2 mapped-position columns)
- mm10 chrom.sizes file
- mm10 filter-regions bed file at 10-kb resolution with MboI (bins with mappability <= 0.8 plus ENCODE blacklist regions)
- Pooled per-subsample contact files for the Juicer HiCCUPS comparator runs

## Outputs

- SnapHiC loop calls per subsample: *.postprocessed.summits.bedpe (plus *.postprocessed.all_candidates.bedpe and allChr.hic/allChr.cool)
- HiCCUPS loop calls per pooled subsample under both default and optimal parameter sets
- loop_counts_subsample_benchmark.csv tabulating loop counts per tool (SnapHiC, HiCCUPS default, HiCCUPS optimal) and subsample size

## How to apply

Assemble per-cell tab-separated (optionally gzipped) contact files for each mES subsample in the 10–742 cell series together with the mm10 chrom.sizes file and the mm10 10-kb MboI filter-regions bed file excluding bins with mappability <= 0.8 and ENCODE blacklist regions. Run SnapHiC (python snap.py under MPI/parallel mode) through all five steps via --steps "bin rwr hic interaction postprocess" at 10-kb binsize with a 2-Mb maximal genomic distance, allocating ~30–40 GB per processor for bin/rwr and ~20 GB per processor for hic/interaction/postprocess and limiting step 3 (combining cells into .hic/.cool files) to ~5 processors per node. Repeat the SnapHiC run independently for every subsample size, then pool each subsample's contacts and run Juicer HiCCUPS twice on the pooled data — once with the default and once with the optimal parameter set. Count loops per tool and subsample size from the *.postprocessed.summits.bedpe files (using fdr_dist as the loop-strength measure) and the HiCCUPS outputs, and compile them into loop_counts_subsample_benchmark.csv. Validate the reproduction against the reported ranges: at 75 cells SnapHiC should recover 1,050–1,420 loops at 10-kb resolution while pooled HiCCUPS yields 0–2 (default) and 3–10 (optimal) loops.

## Related tools

- **SnapHiC** (Runs the five-step loop-calling pipeline (bin, rwr, hic, interaction, postprocess) on each subsample's per-cell contact files to produce the *.postprocessed.summits.bedpe loop calls being benchmarked) — https://github.com/HuMingLab/SnapHiC
- **Juicer** (Supplies the HiCCUPS loop caller that is run on each subsample's pooled contacts with both default and optimal parameter sets as the comparator)
- **cooler** (Converts the SnapHiC combined count matrix into a .cool file during the 'hic' step for each subsample)
- **Dip-C** (Alternative single-cell Hi-C preprocessing pipeline that can generate the per-cell tab-separated contact files consumed by the benchmark) — https://github.com/tanlongzhi/dip-c

## Examples

```
python snap.py --steps 'bin rwr hic interaction postprocess'  # 10-kb binsize, 2-Mb max genomic distance, run per subsample in MPI/parallel mode
```

## Evaluation signals

- At the 75-cell subsample, reproduced SnapHiC counts land within the reported 1,050–1,420 range at 10-kb resolution, while pooled HiCCUPS yields 0–2 loops with default parameters and 3–10 with optimal parameters.
- Every subsample in the 10–742 cell series produces all five SnapHiC step outputs, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe and allChr.hic/allChr.cool.
- loop_counts_subsample_benchmark.csv has one row per subsample size with loop counts for SnapHiC, HiCCUPS-default and HiCCUPS-optimal, tallied using fdr_dist as the loop-strength measure.
- No called loop falls in a filtered bin: the mm10 MboI filter file removes all 10-kb bins with mappability <= 0.8 and all bins overlapping ENCODE blacklist regions.
- Reproduced counts stay within the reported benchmark ranges across the whole series, with SnapHiC loop counts increasing as subsample size grows.

## Limitations

- Pooled-data HiCCUPS recovers almost no loops from low-cell scHi-C subsamples (0–2 default, 3–10 optimal at 75 cells), so the benchmark is highly asymmetric and does not measure per-cell-type loop specificity.
- SnapHiC's default 'inverse' RWR computation is memory-intensive: ~30–40 GB per processor for bin/rwr and ~20 GB for later steps, with step 3 capped at ~5 processors per node; without an HPC the computation can be slow.
- SnapHiC currently accepts only mm10, hg19 and hg38, and the provided filter regions are defined at 10-kb resolution with the MboI restriction enzyme.
- Java >= 8 and MPI are required for .hic generation and parallel runs; documentation only covers SnapHiC v0.2.0 (October 29, 2021) and no changelog is available.

## Evidence

- [abstract] Benchmarking claim for the 742-cell mES series: "Using scHi-C data from 742 mouse embryonic stem cells, we benchmark SnapHiC against a number of computational tools"
- [methods] Five-step pipeline invocation: "use the argument `--step` followed by any of the five options: 'bin', 'rwr', 'hic', 'interaction', and 'postprocess'."
- [methods] Mappability and blacklist filtering threshold: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [methods] Juicer/cooler conversion step: "SnapHiC then uses the Juicer software and the cooler software to convert the count matrix into .hic file and .cool file, respectively."
- [readme] Per-cell contact file format: "In each file, one line represents one contact with 2 columns for chromosome name and 2 columns for the mapped positions (bp)."
- [readme] Output file naming for loop counting: "It is the prefix of the output files, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe, and *allChr.[hic/cool]*."
