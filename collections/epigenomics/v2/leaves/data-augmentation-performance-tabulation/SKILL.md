---
name: data-augmentation-performance-tabulation
description: Use when Hi-C replicate binPairs count matrices are available with FreeHi-C
  augmented versions and a spike-in ground truth, and the goal is to quantify how
  data augmentation affects differential chromatin interaction detection performance.
when_to_use_negative:
- Input is raw FASTQ with no trained FreeHi-C model — this skill evaluates augmented
  differential detection, not the FreeHi-C train/simulate/postProcess pipeline itself
- No spike-in ground truth (no simulated true differential interactions) exists, since
  precision, power, and empirical FDR cannot be computed without it
- The goal is Hi-C data fidelity assessment or contact-map simulation quality rather
  than differential interaction detection benchmarking
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0092
tools:
- name: FreeHi-C
  role: Simulates augmented Hi-C data from interacting genome fragments and provides
    the spike-in module supplying ground truth differential interactions
  repo: https://github.com/yezhengSTAT/FreeHiC
- name: diffHic
  role: Differential chromatin interaction detection method run on original and augmented
    replicate pairs
- name: HiCcompare
  role: Alternative differential chromatin interaction detection method run on original
    and augmented replicate pairs
- name: python3
  role: Environment for computing precision, power, and empirical FDR from detection
    calls against spike-in ground truth
- name: numpy
  role: Python module required for the metric computation environment
- name: scipy
  role: Python module required for the metric computation environment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41592-019-0624-3
    title: FreeHi-C simulates high-fidelity Hi-C data for benchmarking and data augmentation
  source_content_hash: sha256:8938ad35e082588f8fffac4c5f2e6ed600d1804c17520b8693e9b4183d93ef87
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# data-augmentation-performance-tabulation

## Summary

Benchmarks whether augmenting replicate Hi-C data with FreeHi-C simulated data boosts precision and power of differential chromatin interaction detection while preserving false discovery rate control. Produces a side-by-side precision/power/empirical-FDR table comparing original-only vs. augmented detection runs.

## When to use

Use when Hi-C replicate binPairs count matrices are available with FreeHi-C augmented versions and a spike-in ground truth, and the goal is to quantify how data augmentation affects differential chromatin interaction detection performance.

## When NOT to use

- Input is raw FASTQ with no trained FreeHi-C model — this skill evaluates augmented differential detection, not the FreeHi-C train/simulate/postProcess pipeline itself
- No spike-in ground truth (no simulated true differential interactions) exists, since precision, power, and empirical FDR cannot be computed without it
- The goal is Hi-C data fidelity assessment or contact-map simulation quality rather than differential interaction detection benchmarking

## Inputs

- Binned interaction count matrices (binPairs) from original biological Hi-C replicate runs
- Binned interaction count matrices (binPairs) from FreeHi-C simulated/augmented replicate runs (train/simulate/postProcess outputs)
- FreeHi-C spike-in module ground truth (simulated true differential chromatin interactions)
- Nominal FDR level at which differential calls are made in both settings

## Outputs

- differential_detection_metrics.csv with rows for original and augmented conditions and columns precision, power, empirical FDR
- Comparison table of detection performance with and without FreeHi-C augmentation

## How to apply

Load binned interaction count matrices (binPairs) from the original biological replicate runs and from the FreeHi-C simulated/augmented runs (produced via the train/simulate/postProcess pipeline, with simuN set to the targeted sequencing depth and lowerBound set to 2x the resolution to retain valid long-range interactions). Run a differential chromatin interaction detection method (diffHic or HiCcompare) separately on the original replicate pairs and on the augmented replicate pairs, calling interactions at the same nominal FDR level in both settings. Use the FreeHi-C spike-in module ground truth — simulated true differential chromatin interactions — to compute precision (fraction of detected interactions that are true), power (fraction of true differential interactions recovered), and empirical FDR (fraction of false discoveries among calls) for both conditions. Assemble precision, power, and empirical FDR for both conditions into a single comparison table. The rationale is FreeHi-C's demonstrated ability to augment Hi-C data so detection methods gain power without inflating false discoveries.

## Related tools

- **FreeHi-C** (Simulates augmented Hi-C data from interacting genome fragments and provides the spike-in module supplying ground truth differential interactions) — https://github.com/yezhengSTAT/FreeHiC
- **diffHic** (Differential chromatin interaction detection method run on original and augmented replicate pairs)
- **HiCcompare** (Alternative differential chromatin interaction detection method run on original and augmented replicate pairs)
- **python3** (Environment for computing precision, power, and empirical FDR from detection calls against spike-in ground truth)
- **numpy** (Python module required for the metric computation environment)
- **scipy** (Python module required for the metric computation environment)

## Examples

```
git clone https://github.com/yezhengSTAT/FreeHiC
```

## Evaluation signals

- differential_detection_metrics.csv contains rows for both original and augmented conditions
- The CSV has columns for precision, power, and empirical FDR
- The augmented condition shows precision and power gains over the original-only condition
- Empirical FDR in the augmented condition remains controlled at the nominal level
- Simulation inputs were generated with lowerBound set to 2x the resolution for valid long-range interactions and simuN set to the targeted sequencing depth

## Limitations

- bedtools version must be <=2.25.0 (the README warns not to use the latest version)
- The lowerBound filter (2x resolution) removes short-range interactions, so only valid long-range interactions are evaluated
- Evaluation depends entirely on the spike-in module ground truth; without simulated true differential interactions, empirical precision/power/FDR cannot be measured
- FreeHi-C requires real Hi-C sequencing data in FASTQ format as input; it augments deviations from real data and cannot simulate without trained empirical interaction distributions

## Evidence

- [abstract] Augmented FDR-controlled detection boosts precision and power: "FreeHi-C boosts the precision and power of differential chromatin interaction detection through data augmentation under preserved false discovery rate control."
- [methods] lowerBound filter parameter: "lowerBound    : The lower bound for valid long-range interactions. We recommend using 2 times the resolution to remove short-range interactions."
- [methods] Spike-in module provides ground truth: "a spike-in module is added enabling the simulation of true differential chromatin interactions."
- [readme] Simulated output format: "FreeHi-C will eventually provide the simulated genomics contact counts in a sparse matrix format (BED format)"
- [readme] Simulation fidelity rationale: "Data from FreeHi-C exhibit higher fidelity to the biological Hi-C data."
