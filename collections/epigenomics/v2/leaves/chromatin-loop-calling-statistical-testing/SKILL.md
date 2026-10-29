---
name: chromatin-loop-calling-statistical-testing
description: Use when input is per-cell scHi-C or sn-m3C-seq contact files (e.g. 2,869
  human prefrontal cortex sn-m3C-seq cells or 742 mouse ESCs) and the goal is to call
  cell-type-specific chromatin loops at 10-kb resolution.
when_to_use_negative:
- Input is bulk Hi-C or a pooled contact matrix without per-cell contact files — SnapHiC's
  RWR imputation and per-cell binning require one contact file per single cell
- Reference genome is not mm10, hg19, or hg38 — SnapHiC uses the genome name to determine
  the number of autosomal chromosomes and to generate .hic files
- Input is already a per-cell RNA-seq/ATAC-seq table or a Dip-C seg/3dg output — this
  skill only covers chromatin loop calling, not 3D genome reconstruction or preprocessing
  pipelines
edam_operation: http://edamontology.org/operation_0285
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SnapHiC
  role: Five-step loop-calling pipeline (bin, rwr, hic, interaction, postprocess)
    executed via snap.py
  repo: https://github.com/HuMingLab/SnapHiC
- name: Dip-C
  role: Alternative single-cell Hi-C preprocessing pipeline that can generate the
    per-cell contact files SnapHiC consumes
  repo: https://github.com/tanlongzhi/dip-c
- name: Juicer
  role: Converts the combined count matrix into .hic files for Juicebox visualization
    (skippable with --no-hic)
- name: cooler
  role: Converts the combined count matrix into .cool files (skippable with --no-cool)
- name: MPI (open-mpi)
  role: Enables parallel execution of bin/RWR steps on HPC nodes with job schedulers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:5a31c88674d38536a80bdbf01f5e47ff5254f105954dcada21e1377177af1157
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
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_0285
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chromatin-loop-calling-statistical-testing

## Summary

Identify chromatin loops at high resolution from single-cell Hi-C (scHi-C) data using SnapHiC's five-step pipeline—binning, random-walk-with-restart (RWR) imputation, .hic/.cool generation, global/local background loop-candidate testing, and postprocessing into loop summits.

## When to use

Use when input is per-cell scHi-C or sn-m3C-seq contact files (e.g. 2,869 human prefrontal cortex sn-m3C-seq cells or 742 mouse ESCs) and the goal is to call cell-type-specific chromatin loops at 10-kb resolution.

## When NOT to use

- Input is bulk Hi-C or a pooled contact matrix without per-cell contact files — SnapHiC's RWR imputation and per-cell binning require one contact file per single cell
- Reference genome is not mm10, hg19, or hg38 — SnapHiC uses the genome name to determine the number of autosomal chromosomes and to generate .hic files
- Input is already a per-cell RNA-seq/ATAC-seq table or a Dip-C seg/3dg output — this skill only covers chromatin loop calling, not 3D genome reconstruction or preprocessing pipelines

## Inputs

- Per-cell tab-separated (optionally gzipped) contact files, one line per contact with 2 chromosome-name columns and 2 mapped-position (bp) columns
- chrom.sizes file for the reference genome (mm10, hg19 or hg38; prebuilt files in ext/)
- Binned bed file of excluded regions: 10-kb bins with MboI-based mappability <=0.8 and bins overlapping ENCODE blacklist regions (per-genome files in ext/)
- SnapHiC 0.2.0 settings in run_step1.sh/run_step2.sh: SnapHiC_dir, parallelism, number_of_processors, indir, suffix, outdir, chrs, pos, chrlen, genome, filter_file, prefix

## Outputs

- *.postprocessed.summits.bedpe — clustered loop summits (17 columns including outlier_count and fdr_dist) for quantifying loop strength per cell type
- *.postprocessed.all_candidates.bedpe — all loop candidates after removing singletons
- allChr.hic and/or .cool files (Juicer/cooler format) for visualization in Juicebox
- Per-cell-type loop counts (e.g. roughly 817–27,379 loops per cell type at 10-kb resolution in the human prefrontal cortex dataset)

## How to apply

Place per-cell tab-separated contact files (chromosome columns via `chrs`, mapped-position columns via `pos`) in one input directory together with the hg19/mm10/hg38 chrom.sizes file and the 10-kb filter-regions bed file (bins with mappability <=0.8 plus ENCODE blacklist overlaps, MboI-based) from the `ext` directory. Run Part 1 (`python3 snap.py --steps "bin rwr"`) with the parallel MPI mode (open-mpi) and roughly 30–40 GB memory per processor to bin contacts and impute contact probability via random walk with restart (RWR); validate the RWR step with `utils/validate_rwr.py`, delete corrupted files listed in missing.txt, and restart (SnapHiC skips already-processed chromosomes/cells). Run Part 2 (`--steps "hic interaction postprocess"`) with >=20 GB per processor and no more than 5 processors per node to combine cells, identify loop candidates against both global and local background models, and cluster candidates into loop summits; pass `--no-hic`/`--no-cool` if Java/Juicer/cooler outputs are not needed. By default RWR is solved by inverting a chromosome-sized square matrix (method='inverse'), which is memory-heavy; the sliding_window alternative trades accuracy for a smaller footprint. Judge loop strength using fdr_dist in the *.postprocessed.summits.bedpe file, which must contain all 17 required columns including outlier_count and fdr_dist.

## Related tools

- **SnapHiC** (Five-step loop-calling pipeline (bin, rwr, hic, interaction, postprocess) executed via snap.py) — https://github.com/HuMingLab/SnapHiC
- **Dip-C** (Alternative single-cell Hi-C preprocessing pipeline that can generate the per-cell contact files SnapHiC consumes) — https://github.com/tanlongzhi/dip-c
- **Juicer** (Converts the combined count matrix into .hic files for Juicebox visualization (skippable with --no-hic))
- **cooler** (Converts the combined count matrix into .cool files (skippable with --no-cool))
- **MPI (open-mpi)** (Enables parallel execution of bin/RWR steps on HPC nodes with job schedulers)

## Examples

```
python3 snap.py --steps "bin rwr"  # then: python3 snap.py --steps "hic interaction postprocess"
```

## Evaluation signals

- Every output *.postprocessed.summits.bedpe contains all 17 required columns, including outlier_count and fdr_dist
- Tallied loop-summit counts per cell type reproduce the paper's reported range of roughly 817–27,379 loops per cell type at 10-kb resolution on the 2,869-cell human prefrontal cortex sn-m3C-seq dataset
- `python utils/validate_rwr.py <outdir>` reports no corrupted chromosome/cell files, or all files listed in missing.txt are deleted before the RWR restart succeeds
- Low-mappability and blacklist bins are excluded: loop calls avoid the 10-kb bins defined in the provided filter_file (mappability <=0.8, ENCODE blacklist overlaps)
- Run completes with the documented resource envelope (30–40 GB per processor for bin/RWR; >=20 GB per processor and <=5 processors per node for interaction/postprocess)

## Limitations

- Default inverse-matrix RWR requires very large memory (matrix the size of the binned chromosome) and can be slow; the sliding_window approximation is a memory-saving alternative
- SnapHiC currently accepts only mm10, hg19 and hg38 reference genomes and MboI-based mappability filters shipped in ext/
- Java >= 8 is needed for .hic generation; if Java/Juicer/cooler are unavailable, only the --no-hic/--no-cool path is feasible
- Article-supplied enrichment notes no changelog or discussion-section limitations were available, so additional caveats cannot be traced from the provided text

## Evidence

- [methods] SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR): "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR)"
- [methods] global and local background models: "identifying loop candidates based on both global and local background models"
- [methods] mappability and blacklist filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] contact file format: "In each file, one line represents one contact with 2 columns for chromosome name and 2 columns for the mapped positions (bp)."
- [readme] filter regions provided: "i.e., bins with mappability <=0.8 and bins overlapped with the ENCODE blacklist regions, are included in the `ext` directory"
- [readme] Java needed for .hic: "Java is required to generate .hic files for visualization in Juicebox."
