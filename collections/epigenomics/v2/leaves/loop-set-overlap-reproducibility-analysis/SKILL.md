---
name: loop-set-overlap-reproducibility-analysis
description: 'Use when comparing the reproducibility of two loop-calling methods or
  parameter settings on scHi-C data: split per-cell contact files into disjoint replicate
  halves, call loops per half, and quantify symmetric loop-set overlap.'
when_to_use_negative:
- Bulk Hi-C data with no per-cell partitioning possible — the split-half design requires
  many single-cell contact files to build equal replicate groups
- Evaluating loop precision against orthogonal validation data (e.g. ChIP-seq/CTCF
  motifs) rather than replicate reproducibility — overlap analysis does not measure
  biological accuracy
- Comparing callers across different resolutions or genomes (e.g. 10KB mm10 vs hg38
  25KB) without re-running both halves, since filter-regions and bin size must match
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_3949
tools:
- name: SnapHiC
  role: Primary loop caller run per replicate half via snap.py steps 'bin rwr' then
    'hic interaction postprocess'; its .postprocessed.summits.bedpe output supplies
    the SnapHiC loop sets
  repo: https://github.com/HuMingLab/SnapHiC
- name: Juicer
  role: Generates .hic count matrices per half from pooled cells and runs HiCCUPS
    (default and optimal parameters) to produce the comparison loop sets
- name: cooler
  role: Alternative to Juicer for converting the combined count matrix into a .cool
    file for visualization
- name: Juicebox
  role: Visualization of the .hic files generated per replicate half
- name: MPI (open-mpi)
  role: Enables parallel SnapHiC execution across processors/nodes on an HPC cluster
    for each half
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:956457fdb618be4598abde1694e7a20fd6f5b462563085baba7f0ecad52618cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HuMingLab/SnapHiC
  tools:
  - Java
  - Juicebox
  - Juicer
  - MPI (open-mpi)
  - Python
  - SnapHiC
  - cooler
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_3949
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# loop-set-overlap-reproducibility-analysis

## Summary

Benchmarks chromatin loop callers (SnapHiC vs Juicer HiCCUPS) by splitting a single-cell Hi-C population (742 mES cells) into two equal replicate halves and measuring the symmetric fraction of loops recovered between the two independently called loop sets.

## When to use

Use when comparing the reproducibility of two loop-calling methods or parameter settings on scHi-C data: split per-cell contact files into disjoint replicate halves, call loops per half, and quantify symmetric loop-set overlap.

## When NOT to use

- Bulk Hi-C data with no per-cell partitioning possible — the split-half design requires many single-cell contact files to build equal replicate groups
- Evaluating loop precision against orthogonal validation data (e.g. ChIP-seq/CTCF motifs) rather than replicate reproducibility — overlap analysis does not measure biological accuracy
- Comparing callers across different resolutions or genomes (e.g. 10KB mm10 vs hg38 25KB) without re-running both halves, since filter-regions and bin size must match

## Inputs

- 742 per-cell scHi-C contact files (tab-separated or tab-separated+gzipped; 2 chromosome columns + 2 mapped-position columns) from mouse embryonic stem cells
- mm10 chrom.sizes file
- mm10 filter-regions bed file (10KB bins with mappability <=0.8 plus ENCODE blacklist overlaps, MboI-based)
- Java >= 8 and Juicer for .hic generation

## Outputs

- Per-half SnapHiC loop summit sets (*.postprocessed.summits.bedpe, using fdr_dist as loop strength)
- Per-half HiCCUPS loop sets (default and optimal parameters) from each half's .hic file
- reproducibility_summary.csv with symmetric overlap values for SnapHiC, HiCCUPS-default, and HiCCUPS-optimal

## How to apply

Randomly partition the per-cell contact files (tab-separated chromosome/position quadruples; here 742 mES cells) into two disjoint groups of 371 cells each. Run SnapHiC per half using snap.py with --step 'bin rwr' followed by 'hic interaction postprocess' on 10KB bins with the mm10 chrom.sizes and mm10 filter-regions file (bins with mappability <=0.8 and ENCODE blacklist overlaps removed), in parallel mode on an HPC cluster. For each half, also generate .hic count matrices via Juicer and run Juicer HiCCUPS with both default and optimal parameter settings. Extract loop summits from each run's *.postprocessed.summits.bedpe file using fdr_dist as the loop strength measure. For each method/parameter set, compute the symmetric overlap (fraction of half-A loops recovered in half B and vice versa) between the two loop call sets; correct application should yield roughly 50.8% for SnapHiC versus 38.7% (HiCCUPS default) and 39.7% (HiCCUPS optimal), demonstrating SnapHiC's higher reproducibility from 371-cell subsamples.

## Related tools

- **SnapHiC** (Primary loop caller run per replicate half via snap.py steps 'bin rwr' then 'hic interaction postprocess'; its .postprocessed.summits.bedpe output supplies the SnapHiC loop sets) — https://github.com/HuMingLab/SnapHiC
- **Juicer** (Generates .hic count matrices per half from pooled cells and runs HiCCUPS (default and optimal parameters) to produce the comparison loop sets)
- **cooler** (Alternative to Juicer for converting the combined count matrix into a .cool file for visualization)
- **Juicebox** (Visualization of the .hic files generated per replicate half)
- **MPI (open-mpi)** (Enables parallel SnapHiC execution across processors/nodes on an HPC cluster for each half)

## Examples

```
Edit run_step1.sh/run_step2.sh: parallelism=parallel, genome=mm10, chrlen=ext/mm10.chrom.sizes, filter_file=ext/mm10_filter_regions.txt, prefix=mESC_half1; then ./run_step1.sh && ./run_step2.sh
```

## Evaluation signals

- SnapHiC symmetric overlap between the two 371-cell halves approximates the published 50.8%
- HiCCUPS-default symmetric overlap approximates 38.7% and HiCCUPS-optimal approximates 39.7%, i.e. below SnapHiC
- reproducibility_summary.csv contains exactly three reproducibility values (SnapHiC, HiCCUPS default, HiCCUPS optimal)
- Each half's *.postprocessed.summits.bedpe is non-empty and loops use fdr_dist as the strength measure
- The two halves are disjoint (371 + 371 = 742 cells, no cell in both groups)

## Limitations

- Reproducibility measured on only one random split; the workflow does not describe repeating the partition across multiple seeds
- HiCCUPS default settings are tuned for bulk Hi-C, disadvantaging it on 371-cell subsets even with 'optimal' parameter tuning
- SnapHiC 0.2.0 (Oct 29, 2021) accepts only mm10, hg19, hg38 and requires mappability/filter-region files per resolution (10KB/MboI provided); other genomes/resolutions need user-supplied genomic features
- The inverse-matrix RWR computation is memory-hungry and slow; the sliding_window alternative trades accuracy for feasibility on constrained hardware

## Evidence

- [other] split-reproducibility workflow: [evidence span withheld — anchor status: not_found]
- [readme] snap.py steps: "use the argument `--step` followed by any of the five options: 'bin', 'rwr', 'hic', 'interaction', and 'postprocess'."
- [methods] mappability filter: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [methods] HiCCUPS .hic generation: "SnapHiC then uses the Juicer software and the cooler software to convert the count matrix into .hic file and .cool file, respectively."
- [readme] summits output: "the prefix of the output files, including *.postprocessed.summits.bedpe, *.postprocessed.all_candidates.bedpe, and *allChr.[hic/cool]"
