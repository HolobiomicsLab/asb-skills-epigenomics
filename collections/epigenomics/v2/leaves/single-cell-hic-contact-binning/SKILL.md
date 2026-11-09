---
name: single-cell-hic-contact-binning
description: Use when starting from per-cell tab-separated single-cell Hi-C contact
  files (chromosome and mapped-position columns) that must be converted to binned
  contact matrices at 10-kb resolution prior to imputation or loop calling.
when_to_use_negative:
- Input is bulk Hi-C or already-binned .hic/.cool contact matrices rather than per-single-cell
  raw contact lists
- The goal is diploid/haplotype-resolved 3D genome reconstruction (use Dip-C or hickit
  instead of SnapHiC binning)
- A resolution other than 10 kb is required without regenerated mappability/filter
  files — the provided filter files cover mm10, hg19 and hg38 only at 10-kb resolution
  with MboI
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: SnapHiC
  role: Executes the binning step ('--steps "bin"') that aggregates per-cell contacts
    into 10-kb bins and applies the mappability/blacklist filter file
  repo: https://github.com/HuMingLab/SnapHiC
- name: Dip-C
  role: Optional upstream single-cell Hi-C preprocessing pipeline that can generate
    the per-cell contact files consumed by the binning step
  repo: https://github.com/tanlongzhi/dip-c
- name: MPI (open-mpi)
  role: Parallelization backend for running the bin step across chromosomes and cells
    on an HPC cluster
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:9c7828156341a7fabac8f117cb45deeb1abb44ee4d88a12ec30ed1126d44acbf
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
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-hic-contact-binning

## Summary

Bins per-single-cell mapped read-pair contacts into fixed 10-kb genomic bins as the first step of the SnapHiC pipeline, excluding unreliable bins (mappability <=0.8 and ENCODE blacklist overlaps) before downstream RWR imputation and loop calling.

## When to use

Use when starting from per-cell tab-separated single-cell Hi-C contact files (chromosome and mapped-position columns) that must be converted to binned contact matrices at 10-kb resolution prior to imputation or loop calling.

## When NOT to use

- Input is bulk Hi-C or already-binned .hic/.cool contact matrices rather than per-single-cell raw contact lists
- The goal is diploid/haplotype-resolved 3D genome reconstruction (use Dip-C or hickit instead of SnapHiC binning)
- A resolution other than 10 kb is required without regenerated mappability/filter files — the provided filter files cover mm10, hg19 and hg38 only at 10-kb resolution with MboI

## Inputs

- Per-cell tab-separated (or tab-separated and gzipped) contact files, one line per contact with 2 chromosome columns and 2 mapped-position (bp) columns
- chrom.sizes file for the reference genome (mm10, hg19, or hg38)
- Binned bed filter-regions file (bins with mappability <=0.8 and ENCODE blacklist overlaps) at 10-kb resolution with MboI, from the ext directory

## Outputs

- Per-chromosome, per-cell 10-kb-binned contact count matrices with low-mappability and blacklisted bins removed (input to the RWR imputation step)

## How to apply

Place one tab-separated (optionally gzipped) contact file per cell in a single input directory, with two columns for chromosome names and two columns for mapped positions, and point SnapHiC to them via the indir, chrs, and pos variables in run_step1.sh. Provide a chrom.sizes file for the reference genome (mm10, hg19, or hg38, included in the ext directory) and a binned bed filter file of excluded regions — bins with mappability <=0.8 based on the MboI restriction enzyme plus all 10-kb bins overlapping ENCODE blacklist regions. Run the 'bin' step of snap.py (typically combined with the RWR step as 'bin rwr') under the parallel (open-mpi), threaded, or single-proc mode, allocating roughly 30-40 GB memory per processor for the binning/imputation phase. Contacts are aggregated into 10-kb bins per chromosome per cell, and filtered bins are removed so that only mappable, non-blacklisted bins enter the contact probability matrix. Contact files may be produced upstream by preprocessing pipelines such as Dip-C. Binning quality directly determines the fidelity of the subsequent random-walk-with-restart imputation and loop-candidate testing.

## Related tools

- **SnapHiC** (Executes the binning step ('--steps "bin"') that aggregates per-cell contacts into 10-kb bins and applies the mappability/blacklist filter file) — https://github.com/HuMingLab/SnapHiC
- **Dip-C** (Optional upstream single-cell Hi-C preprocessing pipeline that can generate the per-cell contact files consumed by the binning step) — https://github.com/tanlongzhi/dip-c
- **MPI (open-mpi)** (Parallelization backend for running the bin step across chromosomes and cells on an HPC cluster)

## Examples

```
python3 snap.py --steps "bin rwr"
```

## Evaluation signals

- Each input cell yields binned output for every autosomal chromosome of the specified genome, with bins overlapping the filter file (mappability <=0.8 or ENCODE blacklist) absent from the matrices
- Column indices given by chrs and pos correctly resolve chromosome and position fields in every contact file (e.g. chrs='2 4', pos='3 5'); mis-specified columns produce empty or malformed bins
- The bin count matches the expected number of 10-kb bins from chrom.sizes minus filtered regions for the chosen genome (mm10/hg19/hg38)
- The subsequent RWR validation (utils/validate_rwr.py listing corrupted chromosome/cell files in missing.txt) reports no missing entries attributable to the binning step
- Downstream loop calling on the ODC_100 test dataset reproduces the reference 6,249 loop summits, which depends on correct binning upstream

## Limitations

- SnapHiC currently accepts only mm10, hg19 and hg38 reference genomes, and the bundled filter regions are defined at 10-kb resolution with the MboI restriction enzyme; other resolutions or enzymes require downloading external genomic-feature resources
- The default binning/RWR path requires large memory (roughly 30-40 GB per processor), and corrupted chromosome/cell outputs must be detected and the step restarted (SnapHiC skips already-processed chromosomes/cells)
- All bins with mappability <=0.8 and ENCODE blacklist overlaps are discarded, so loops involving these regions cannot be detected

## Evidence

- [methods] SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR): "SnapHiC consists of five steps: (1) binning, (2) imputing contact probability via random walk with restart (RWR)"
- [methods] We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions.: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] one line represents one contact with 2 columns for chromosome name and 2 columns for the mapped positions (bp).: "In each file, one line represents one contact with 2 columns for chromosome name and 2 columns for the mapped positions (bp)."
- [readme] Filtered regions for mm10, hg19 and hg38 at 10KB resolution with the restriction enzyme MboI, i.e., bins with mappability <=0.8 and bins overlapped with the ENCODE blacklist regions, are included in the `ext` directory.: "Filtered regions for mm10, hg19 and hg38 at 10KB resolution with the restriction enzyme MboI, i.e., bins with mappability <=0.8 and bins overlapped with the ENCODE blacklist regions, are included in"
- [other] Run Part 1 of SnapHiC with python3 snap.py --steps "bin rwr" to bin contacts into 10-kb bins: [evidence span withheld — anchor status: not_found]
