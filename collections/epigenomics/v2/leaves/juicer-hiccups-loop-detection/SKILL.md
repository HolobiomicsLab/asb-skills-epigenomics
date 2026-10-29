---
name: juicer-hiccups-loop-detection
description: 'Use when calling or benchmarking chromatin loops from pooled scHi-C
  contact matrices: run Juicer HiCCUPS on .hic files built from ~371-cell mESC halves
  at 10KB resolution under both default and optimal parameters, then score replicate
  overlap against SnapHiC.'
when_to_use_negative:
- Only raw per-cell contact lists are available with no aggregated .hic matrix — HiCCUPS
  runs on .hic files, so the SnapHiC 'bin rwr' + 'hic' steps must build the pooled
  matrix first
- Loops must be called from individual sparse scHi-C cells without pooling — the paper
  benchmarks SnapHiC (50.8% reproducibility) as superior to pooled HiCCUPS (38.7–39.7%)
  on such data
- The goal is cell-type-specific loop discovery from single-nucleus methyl-3C-seq
  (e.g., 2,869 human prefrontal cortical cells) — the paper applies SnapHiC there,
  not the HiCCUPS baseline
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: Juicer
  role: Converts each half's combined 10KB-binned count matrix into allChr.hic and
    supplies the HiCCUPS kernel-based loop caller, executed once with default and
    once with optimal parameters
- name: SnapHiC
  role: Upstream pipeline (snap.py steps 'bin rwr' + 'hic interaction postprocess')
    that produces the .hic matrices and the *.postprocessed.summits.bedpe loop summits
    against which HiCCUPS calls are benchmarked
  repo: github.com/HuMingLab/SnapHiC
- name: Juicebox
  role: Visualization target for the .hic contact matrices on which HiCCUPS loop calls
    are made
- name: Java
  role: Runtime dependency (version >= 8) needed to generate .hic files for the HiCCUPS/Juicebox
    arm of the benchmark
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-021-01231-2
    title: 'SnapHiC: a computational pipeline to identify chromatin loops from single-cell
      Hi-C data'
  source_content_hash: sha256:4fae404145cb860506ae97b9fb59038dfa1d852ff8394bdfad644757ebeaae95
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - Juicebox
  - Juicer
  - MPI (open-mpi)
  - Python
  - SnapHiC
  - cooler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# juicer-hiccups-loop-detection

## Summary

Runs Juicer's HiCCUPS loop caller on pseudo-bulk .hic contact matrices built from pooled single-cell Hi-C contacts; in the SnapHiC paper it is the pooled-baseline benchmark run on two 371-cell mESC halves (mm10, 10KB bins) with default and optimal parameters, yielding 38.7%/39.7% replicate loop reproducibility versus SnapHiC's 50.8%.

## When to use

Use when calling or benchmarking chromatin loops from pooled scHi-C contact matrices: run Juicer HiCCUPS on .hic files built from ~371-cell mESC halves at 10KB resolution under both default and optimal parameters, then score replicate overlap against SnapHiC.

## When NOT to use

- Only raw per-cell contact lists are available with no aggregated .hic matrix — HiCCUPS runs on .hic files, so the SnapHiC 'bin rwr' + 'hic' steps must build the pooled matrix first
- Loops must be called from individual sparse scHi-C cells without pooling — the paper benchmarks SnapHiC (50.8% reproducibility) as superior to pooled HiCCUPS (38.7–39.7%) on such data
- The goal is cell-type-specific loop discovery from single-nucleus methyl-3C-seq (e.g., 2,869 human prefrontal cortical cells) — the paper applies SnapHiC there, not the HiCCUPS baseline

## Inputs

- Tab-separated per-cell scHi-C contact files (2 chromosome + 2 mapped-position columns) from 742 mouse embryonic stem cells, partitioned into two disjoint 371-cell halves
- Pseudo-bulk .hic count matrix per half (allChr.hic) generated via Juicer (requires Java >= 8)
- mm10 chrom.sizes and mm10 filter-regions file (10KB bins with MboI mappability <=0.8 and ENCODE blacklist overlaps)
- HiCCUPS parameter settings: default and paper-optimal

## Outputs

- HiCCUPS loop call sets (BEDPE-style loop lists) per 371-cell half, for both default and optimal parameter settings
- SnapHiC loop summits (*.postprocessed.summits.bedpe, fdr_dist as loop-strength measure) for the comparison arm
- reproducibility_summary.csv with symmetric-overlap reproducibility for SnapHiC, HiCCUPS-default, and HiCCUPS-optimal (targets 50.8%, 38.7%, 39.7%)

## How to apply

Pool the tab-separated per-cell contact files (2 chromosome + 2 mapped-position columns) of each 371-cell half of the 742 mESC dataset and convert the combined 10KB-binned count matrix to .hic with Juicer (requires Java >= 8; upstream SnapHiC steps 'bin rwr' then 'hic interaction postprocess' use mm10 chrom.sizes and mm10 filter regions excluding MboI mappability <=0.8 bins and ENCODE blacklist overlaps, under MPI/open-mpi on an HPC). Run Juicer HiCCUPS on each half's allChr.hic twice — once with default parameters and once with the paper's optimal parameter settings — producing two loop call sets per replicate. Extract loop coordinates/summits (SnapHiC's *.postprocessed.summits.bedpe uses fdr_dist as the loop-strength measure) and compute the symmetric overlap: the fraction of half-A loops recovered in half B and vice versa. Judge success by agreement with the published benchmark: HiCCUPS default 38.7%, HiCCUPS optimal 39.7%, versus SnapHiC 50.8%.

## Related tools

- **Juicer** (Converts each half's combined 10KB-binned count matrix into allChr.hic and supplies the HiCCUPS kernel-based loop caller, executed once with default and once with optimal parameters)
- **SnapHiC** (Upstream pipeline (snap.py steps 'bin rwr' + 'hic interaction postprocess') that produces the .hic matrices and the *.postprocessed.summits.bedpe loop summits against which HiCCUPS calls are benchmarked) — github.com/HuMingLab/SnapHiC
- **Juicebox** (Visualization target for the .hic contact matrices on which HiCCUPS loop calls are made)
- **Java** (Runtime dependency (version >= 8) needed to generate .hic files for the HiCCUPS/Juicebox arm of the benchmark)

## Examples

```
bash run_step1.sh && bash run_step2.sh
```

## Evaluation signals

- Symmetric overlap between the two 371-cell halves approaches the published 38.7% (HiCCUPS default) and 39.7% (HiCCUPS optimal), versus 50.8% for SnapHiC
- HiCCUPS-optimal reproducibility should exceed HiCCUPS-default (39.7% vs 38.7%); a reversed gap indicates a mis-set parameter run
- All called loops fall on mm10 10KB bins outside the filter regions (mappability <=0.8 bins and ENCODE blacklist overlaps)
- Upstream allChr.hic files open in Juicebox (Java >= 8) and derive from the pooled tab-separated per-cell contact lists
- reproducibility_summary.csv contains exactly the three method entries: SnapHiC, HiCCUPS-default, HiCCUPS-optimal

## Limitations

- On 371-cell mESC halves HiCCUPS recovers fewer reproducible loops than SnapHiC (38.7% default / 39.7% optimal vs 50.8%) — the paper's core benchmark result
- Tuning HiCCUPS parameters yields only ~1 percentage-point gain (39.7% vs 38.7%), so parameter optimization does not close the gap to SnapHiC
- Requires Java >= 8 for .hic generation and an HPC with MPI/open-mpi and ample memory for the upstream SnapHiC matrix step; HiCCUPS operating on pooled matrices loses signal at scHi-C subsample sizes
- Benchmark values are specific to the 742-cell mm10 mESC dataset at 10KB resolution with MboI-based mappability filtering and may not transfer to other genomes or resolutions

## Evidence

- [methods] Juicer conversion of pooled contacts: "SnapHiC then uses the Juicer software and the cooler software to convert the count matrix into .hic file and .cool file, respectively."
- [methods] Java requirement for .hic generation: "Install Java version >= 8 and include it in your path. Java is required to generate .hic files for visualization in Juicebox."
- [methods] Step-based pipeline control: "use the argument `--step` followed by any of the five options: 'bin', 'rwr', 'hic', 'interaction', and 'postprocess'."
- [readme] Mappability and blacklist filtering: "We removed all 10KB bins with mappability <=0.8, and all 10KB bins overlapped with the ENCODE blacklist regions."
- [readme] HPC recommendation: "We strongly recommend using an HPC environment where you can request multiple nodes/processors and allocate memory."
- [other] Published reproducibility benchmark: [evidence span withheld — anchor status: not_found]
