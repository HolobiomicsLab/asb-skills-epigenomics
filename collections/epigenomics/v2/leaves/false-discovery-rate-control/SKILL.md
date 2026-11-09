---
name: false-discovery-rate-control
description: Use when comparing differential chromatin interaction detection performance
  (precision, power, empirical FDR) between original Hi-C replicate pairs and FreeHi-C-augmented
  replicates called at the same nominal FDR level.
when_to_use_negative:
- You only have a pre-built contact matrix with no raw FASTQ — FreeHi-C requires real
  Hi-C sequencing data (FASTQ format) as input for training, so matrix-only input
  cannot be augmented.
- Your goal is plain differential interaction calling in real replicates, not benchmarking
  the precision/power/FDR trade-off against spike-in ground truth.
- Non-Hi-C differential detection tasks (e.g., RNA-seq) where the spike-in module
  and binPairs machinery do not apply.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FreeHi-C
  role: Data-driven Hi-C simulator (v2.0) that augments biological replicates with
    simulated replicates and provides the spike-in module supplying ground truth differential
    chromatin interactions for empirical FDR estimation.
  repo: https://github.com/yezhengSTAT/FreeHiC
- name: diffHic
  role: Differential chromatin interaction detection method run on original replicate
    pairs and FreeHi-C-augmented replicate pairs at the same nominal FDR level.
- name: HiCcompare
  role: Alternative differential detection method used to call interactions at the
    same nominal FDR in both original and augmented settings for comparison.
- name: python3
  role: Runs the metric computation (precision, power, empirical FDR from spike-in
    ground truth) and assembles differential_detection_metrics.csv; numpy (>=1.13.1)
    and scipy (>=0.19.1) are required modules.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0624-3
    title: FreeHi-C simulates high-fidelity Hi-C data for benchmarking and data augmentation
  source_content_hash: sha256:f3a862ffd27304db715c819486babcd039e3bc775356e2dd3a72ed7b8a6f3a21
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/yezhengSTAT/FreeHiC
  tools:
  - FreeHi-C
  - HiCcompare
  - diffHic
  - numpy
  - python3
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# false-discovery-rate-control

## Summary

Augments biological Hi-C replicates with FreeHi-C simulated data so that differential chromatin interaction detection (diffHic or HiCcompare) gains precision and power while empirical false discovery rate stays at the nominal level; performance is quantified against the FreeHi-C spike-in module's known ground truth differential interactions.

## When to use

Use when comparing differential chromatin interaction detection performance (precision, power, empirical FDR) between original Hi-C replicate pairs and FreeHi-C-augmented replicates called at the same nominal FDR level.

## When NOT to use

- You only have a pre-built contact matrix with no raw FASTQ — FreeHi-C requires real Hi-C sequencing data (FASTQ format) as input for training, so matrix-only input cannot be augmented.
- Your goal is plain differential interaction calling in real replicates, not benchmarking the precision/power/FDR trade-off against spike-in ground truth.
- Non-Hi-C differential detection tasks (e.g., RNA-seq) where the spike-in module and binPairs machinery do not apply.

## Inputs

- Raw paired-end Hi-C sequencing FASTQ files per biological replicate (split as *_1.fastq and *_2.fastq)
- Reference genome file and restriction enzyme fragment file (refrag)
- FreeHiC_parameters configuration file (software paths, simuN, lowerBound, train/simulate/postProcess indicators)
- Binned interaction count matrices (binPairs) from original biological replicate runs
- FreeHi-C simulated/augmented binPairs (BED-format sparse matrices of genomic contact counts)
- FreeHi-C spike-in module ground truth: simulated true differential chromatin interactions

## Outputs

- Simulated Hi-C sequencing reads and post-processed binPairs (per-chromosome, BED sparse matrix format)
- differential_detection_metrics.csv with rows for both original and augmented conditions and columns for precision, power, and empirical FDR
- Comparison table of differential chromatin interaction detection performance with vs without FreeHi-C augmentation at a fixed nominal FDR

## How to apply

Process each biological replicate's paired-end FASTQs through the FreeHi-C pipeline (train, then simulate, then postProcess), setting simuN to the targeted sequencing depth and lowerBound to 2 times the resolution so only valid long-range interactions are simulated. Load the binned interaction count matrices (binPairs) from the original biological replicate runs and the FreeHi-C simulated/augmented runs, then run diffHic or HiCcompare separately on the original replicate pairs and on the data-augmented replicate pairs, calling interactions at the same nominal FDR level in both settings. Use the FreeHi-C spike-in module's simulated true differential chromatin interactions as ground truth to compute precision (fraction of detected interactions that are true), power (fraction of true differential interactions recovered), and empirical FDR (fraction of false discoveries among calls). Assemble these metrics for original-only and augmented conditions into a single table such as differential_detection_metrics.csv. The skill is applied correctly when the augmented condition shows precision and power gains while empirical FDR remains controlled at the nominal level.

## Related tools

- **FreeHi-C** (Data-driven Hi-C simulator (v2.0) that augments biological replicates with simulated replicates and provides the spike-in module supplying ground truth differential chromatin interactions for empirical FDR estimation.) — https://github.com/yezhengSTAT/FreeHiC
- **diffHic** (Differential chromatin interaction detection method run on original replicate pairs and FreeHi-C-augmented replicate pairs at the same nominal FDR level.)
- **HiCcompare** (Alternative differential detection method used to call interactions at the same nominal FDR in both original and augmented settings for comparison.)
- **python3** (Runs the metric computation (precision, power, empirical FDR from spike-in ground truth) and assembles differential_detection_metrics.csv; numpy (>=1.13.1) and scipy (>=0.19.1) are required modules.)

## Examples

```
git clone https://github.com/yezhengSTAT/FreeHiC && pip install -r python-requirements.txt
```

## Evaluation signals

- differential_detection_metrics.csv contains rows for both original and augmented conditions
- The metrics table has precision, power, and empirical FDR columns populated for each condition
- The augmented condition shows power and precision gains over the original replicate-only detection
- Empirical FDR of the augmented-condition calls remains at or below the nominal FDR level used for calling
- Simulation used simuN set to the targeted sequencing depth and lowerBound set to 2x the resolution so only valid long-range interactions are simulated

## Limitations

- bedtools must be <=2.25.0; the README explicitly warns not to use the latest bedtools version.
- FreeHi-C requires real Hi-C sequencing data (FASTQ format) with user-defined simulation parameters; it cannot start from already-processed contact matrices.
- Conda environments may hit GLIBCXX errors with bedtools, requiring a libgcc update (conda install -c anaconda libgcc=5.2.0).
- FDR preservation is demonstrated in the spike-in benchmark setting where ground truth differential interactions are simulated; empirical FDR depends on having that known ground truth.

## Evidence

- [abstract] FreeHi-C boosts precision and power of differential chromatin interaction detection through data augmentation while preserving FDR control: "FreeHi-C boosts the precision and power of differential chromatin interaction detection through data augmentation under preserved false discovery rate control."
- [readme] FreeHi-C boosts power and enables FDR control for differential interaction detection via augmentation: "boosts power and enables false discovery rate control for differential interaction detection algorithms through data augmentation"
- [readme] Spike-in module provides known ground truth for comparing differential detection methods: "a comprehensive comparison of differential chromatin interaction detection methods where the ground truth differential chromatin interactions are known"
- [methods] lowerBound parameter for valid long-range interactions: "The lower bound for valid long-range interactions. We recommend using 2 times the resolution to remove short-range interactions."
- [methods] Spike-in module simulates true differential chromatin interactions: "a spike-in module is added enabling the simulation of true differential chromatin interactions."
