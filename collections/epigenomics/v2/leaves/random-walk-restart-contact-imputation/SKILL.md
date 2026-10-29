---
name: random-walk-restart-contact-imputation
description: Use when individual single-cell Hi-C contact files are too sparse for
  direct loop detection and normalized per-cell contact probabilities at 10-kb resolution
  must be imputed before statistical testing.
when_to_use_negative:
- Input data is bulk Hi-C or pooled contacts where per-cell imputation is unnecessary
  and standard loop callers like Juicer HiCCUPS suffice
- Bins or regions fall in the excluded filter file (mappability <=0.8 or ENCODE blacklist
  overlaps), where imputed values should not drive loop calling
- Memory-constrained environments cannot host the chromosome-sized matrix inversion
  and the sliding_window approximation is unacceptable for the analysis
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3168
tools:
- name: SnapHiC
  role: Implements the RWR imputation step ('rwr' in --steps 'bin rwr') over 10-kb
    binned per-chromosome per-cell contact matrices
  repo: https://github.com/HuMingLab/SnapHiC
- name: Dip-C
  role: Alternative single-cell Hi-C preprocessing pipeline that can generate the
    per-cell contact files used as RWR input
  repo: https://github.com/tanlongzhi/dip-c
- name: MPI (open-mpi)
  role: Parallelizes the RWR computation across chromosomes and cells on HPC ('which
    mpicc' must return a path)
  repo: https://www.open-mpi.org/
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:209dab5e8760a419f0ad1a10732bb51afe4b8b11b17e596e60436a77b82dda6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Dip-C
  - Java
  - Java >= 8
  - MPI (open-mpi)
  - Python
  - Python >= 3.6
  - Singularity
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# random-walk-restart-contact-imputation

## Summary

Imputes per-cell contact probabilities in single-cell Hi-C by solving the random walk with restart (RWR) equation over a 10-kb binned contact matrix per chromosome, densifying extremely sparse single-cell data so loop calling becomes statistically tractable.

## When to use

Use when individual single-cell Hi-C contact files are too sparse for direct loop detection and normalized per-cell contact probabilities at 10-kb resolution must be imputed before statistical testing.

## When NOT to use

- Input data is bulk Hi-C or pooled contacts where per-cell imputation is unnecessary and standard loop callers like Juicer HiCCUPS suffice
- Bins or regions fall in the excluded filter file (mappability <=0.8 or ENCODE blacklist overlaps), where imputed values should not drive loop calling
- Memory-constrained environments cannot host the chromosome-sized matrix inversion and the sliding_window approximation is unacceptable for the analysis

## Inputs

- Per-cell tab-separated (optionally gzipped) contact files, one line per contact with 2 chromosome-name columns and 2 mapped-position columns (specified via chrs and pos column indices)
- chrom.sizes file for the reference genome (mm10, hg19, or hg38)
- Binned bed filter file of excluded regions (bins with mappability <=0.8 and ENCODE blacklist overlaps, e.g. ext/mm10_filter_regions.txt)

## Outputs

- Per-chromosome, per-cell imputed contact probability matrices from the RWR step (intermediate files in outdir)
- A missing.txt file listing corrupted chromosome/cell outputs after validation
- Downstream combined count matrix converted to .hic (Juicer) and .cool (cooler) files

## How to apply

After binning contacts into 10-kb bins (removing bins with mappability <=0.8 and ENCODE blacklist overlaps), run the RWR step of SnapHiC per chromosome per cell ('bin rwr' stage) to solve for imputed contact probabilities. By default the linear algebra equation is solved by inverting a large square matrix sized to the binned chromosome, which requires large memory (roughly 30-40 GB per processor recommended) and can be slow; alternatively set the 'method' variable to 'sliding_window' to compute RWR over a small window around each bin at the cost of approximation. Because the step is parallelized via MPI per chromosome and per cell, validate outputs with 'python utils/validate_rwr.py <outdir>', delete corrupted chromosome/cell files listed in missing.txt, and rerun — SnapHiC skips already-processed chromosomes/cells. The imputed per-cell contact probability matrices are then combined across cells and normalized for downstream loop-candidate testing (paired t-test against global and five local background models).

## Related tools

- **SnapHiC** (Implements the RWR imputation step ('rwr' in --steps 'bin rwr') over 10-kb binned per-chromosome per-cell contact matrices) — https://github.com/HuMingLab/SnapHiC
- **Dip-C** (Alternative single-cell Hi-C preprocessing pipeline that can generate the per-cell contact files used as RWR input) — https://github.com/tanlongzhi/dip-c
- **MPI (open-mpi)** (Parallelizes the RWR computation across chromosomes and cells on HPC ('which mpicc' must return a path)) — https://www.open-mpi.org/

## Examples

```
python3 snap.py --steps "bin rwr"  # run_step1.sh variables: genome=hg19, filter_file=ext/hg19_filter_regions.txt, method=inverse or sliding_window
```

## Evaluation signals

- Run 'python utils/validate_rwr.py <outdir>' and confirm missing.txt is empty or that all listed corrupted chromosome/cell files are deleted and the RWR step rerun cleanly
- Imputed contact probability matrices exist for every (cell, chromosome) pair, with SnapHiC skipping already-processed chromosomes/cells on restart
- Downstream postprocessed summits reproduce reference counts — e.g. the ODC_100 test dataset yields 6,249 loops in odc_100.postprocessed.summits.bedpe
- Contact files were first binned at 10-kb resolution with low-mappability (<=0.8) and ENCODE blacklist bins removed before RWR
- Memory allocation matched the method: ~30-40 GB per processor for bin/rwr when using matrix inversion

## Limitations

- Default matrix-inversion RWR over each binned chromosome requires large memory and can take long to compute; the sliding_window alternative trades accuracy for smaller, faster windows
- RWR outputs can be corrupted on HPC failures, requiring validation, deletion of files in missing.txt, and a restart of the step
- SnapHiC currently accepts only mm10, hg19 and hg38 genomes for the pipeline that contains the RWR step

## Evidence

- [methods] Five-step RWR workflow: "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR), (3) combining all cells to generate .hic file"
- [readme] Matrix inversion default: "By default, for the calculation of the random walk values over each chromosome, a linear algebra equation is solved by computing the inverse of a large square matrix (size of the binned chromosome)."
- [readme] Sliding window alternative: "instead of computing the RWR values over the entire chromosome in a single step, a relatively small"
- [methods] Filtering thresholds: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [other] RWR validation utility: [evidence span withheld — anchor status: not_found]
