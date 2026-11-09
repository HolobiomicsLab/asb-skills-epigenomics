---
name: opencl-gpu-runtime-profiling
description: Use when you must quantify whether a network-inference pipeline is fast
  enough for repeated runs on commodity hardware, e.g. timing Genexpi's OpenCL GPU
  core vs TD-Aracne on the SigB and SigR regulon benchmarks.
when_to_use_negative:
- The goal is assessing inference accuracy (fit-quality thresholds, no-change and
  constant-synthesis filtering) rather than compute cost — timing says nothing about
  regulon correctness.
- 'No OpenCL-capable device is available: Genexpi''s compute core (simulated annealing
  and Euler-integrated error evaluation) is implemented in OpenCL, so GPU runtime
  profiling cannot be reproduced.'
- Input is not time-course expression data paired with candidate regulon definitions
  — there is nothing to run through Genexpi or TD-Aracne to time.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Genexpi
  role: OpenCL GPU compute core whose per-regulon wall-clock time is measured (simulated
    annealing, 128 runs per gene and candidate regulator set)
  repo: github.com/cas-bioinf/genexpi
- name: ARACNE
  role: TD-Aracne, the slower network-inference baseline timed on the same three regulon
    datasets for comparison
- name: OpenCL
  role: GPU compute backend implementing the simulated-annealing fits and Euler-integrated
    error evaluation that the profiling targets
- name: R
  role: R package/notebook interface used to run Genexpi and reproduce the evaluation
    (Additional file 1 R notebook)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:ee09587785800617bb160f48611946a0390da5de689d3b6a503716ffd2ea383f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARACNE
  - Genexpi
  - Java
  - OpenCL
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# opencl-gpu-runtime-profiling

## Summary

A runtime-profiling skill that measures and compares wall-clock compute times of Genexpi's OpenCL GPU compute core (simulated annealing with 128 annealing runs per gene) against TD-Aracne. It establishes whether the regulon-inference pipeline is fast enough to run repeatedly on commodity hardware.

## When to use

Use when you must quantify whether a network-inference pipeline is fast enough for repeated runs on commodity hardware, e.g. timing Genexpi's OpenCL GPU core vs TD-Aracne on the SigB and SigR regulon benchmarks.

## When NOT to use

- The goal is assessing inference accuracy (fit-quality thresholds, no-change and constant-synthesis filtering) rather than compute cost — timing says nothing about regulon correctness.
- No OpenCL-capable device is available: Genexpi's compute core (simulated annealing and Euler-integrated error evaluation) is implemented in OpenCL, so GPU runtime profiling cannot be reproduced.
- Input is not time-course expression data paired with candidate regulon definitions — there is nothing to run through Genexpi or TD-Aracne to time.

## Inputs

- Time-course expression data: GSE44415 for the SigR regulons of Streptomyces coelicolor and the B. subtilis SigB benchmark series
- Candidate regulon definitions: SigB regulon from Subtiwiki (as of January 2017), SigR ChIP-chip regulon (Kallifidas et al.), SigR knockout regulon (Kim et al.)
- Expression profiles smoothed by B-spline linear regression (degrees of freedom approximately half the number of time points)
- OpenCL-capable mid-tier GPU for Genexpi's compute core and a runtime environment for TD-Aracne

## Outputs

- Per-regulon wall-clock runtime measurements in seconds for Genexpi (reported 26/109/150 s) and TD-Aracne (reported 447/718/446 s)
- timing_table.csv: comparison table with one row per regulon (SigB, SigR ChIP-chip, SigR knockout) and one column per method (Genexpi, TD-Aracne)

## How to apply

Assemble the three benchmark datasets — the SigB regulon of B. subtilis from Subtiwiki and the two SigR regulons of Streptomyces coelicolor (ChIP-chip from Kallifidas et al. and knockout from Kim et al.) combined with the GSE44415 expression time series — and smooth expression profiles with B-spline linear regression (degrees of freedom approximately half the number of time points) as Genexpi requires before computation. Run Genexpi through its OpenCL compute core on a GPU; the engine minimizes the model error by simulated annealing with 128 annealing runs per gene and candidate regulator set and evaluates the error function via Euler integration on subsampled trajectories — record wall-clock time per regulon (reported: 26 s SigB, 109 s SigR ChIP-chip, 150 s SigR knockout). Run TD-Aracne on the same three regulon datasets, recording per-regulon wall-clock times (reported: 447 s, 718 s, 446 s), keeping hardware fixed (mid-tier GPU) so the comparison is like-for-like. Assemble a comparison table with one row per regulon and one column per method, then validate each measured runtime against the reported values within a sane tolerance band before writing timing_table.csv. Decision point: Genexpi qualifies as fast enough for repeated runs on commodity hardware if measured runtimes stay in the tens-to-hundreds-of-seconds range, while TD-Aracne is slower but still fast enough for most practical use cases.

## Related tools

- **Genexpi** (OpenCL GPU compute core whose per-regulon wall-clock time is measured (simulated annealing, 128 runs per gene and candidate regulator set)) — github.com/cas-bioinf/genexpi
- **ARACNE** (TD-Aracne, the slower network-inference baseline timed on the same three regulon datasets for comparison)
- **OpenCL** (GPU compute backend implementing the simulated-annealing fits and Euler-integrated error evaluation that the profiling targets)
- **R** (R package/notebook interface used to run Genexpi and reproduce the evaluation (Additional file 1 R notebook))

## Evaluation signals

- Each measured Genexpi runtime falls within a sane tolerance band of the reported 26 s (SigB), 109 s (SigR ChIP-chip), and 150 s (SigR knockout).
- TD-Aracne runtimes (reported 447 s, 718 s, 446 s) are measured on the identical three regulon datasets, so the per-regulon comparison is like-for-like.
- timing_table.csv exists with exactly one row per regulon (SigB, SigR ChIP-chip, SigR knockout) and one column per method (Genexpi, TD-Aracne), with no missing cells.
- Expression profiles were smoothed with B-spline linear regression (degrees of freedom approximately half the number of time points) before any timing runs.
- Genexpi ran with the documented 128 annealing runs per gene and candidate regulator set, so timing reflects the paper's compute load.

## Limitations

- Absolute timings are hardware-specific: the comparison was run on a mid-tier GPU, so reported values (26/109/150 s vs 447/718/446 s) do not transfer across devices.
- The article states no software version numbers for Genexpi, R, RStudio, or Cytoscape, so runtime provenance cannot be pinned to a release.
- Runtime scales with candidate target counts; the yeast evaluation reduced computational burden by selecting 30 targets per transcription factor, so timings on unfiltered regulons will differ.

## Evidence

- [methods] Genexpi's compute core executes 128 simulated-annealing runs per gene and candidate regulator set: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs with different initial parameter values."
- [methods] Error-function evaluation is implemented in OpenCL via Euler integration on subsampled trajectories: "The subsampling then allows us to integrate (1) accurately with the computationally cheap Euler method, making evaluation of the error function fast and easy to implement in OpenCL."
- [methods] Profiles must be smoothed with B-spline linear regression (df approx. half the number of time points) before computation: "Genexpi encourages smoothing the data prior to computation. We have had good results with linear regression of B-spline basis with degrees of freedom equal to approximately half the number of"
- [readme] Documentation and binaries live in the project's wiki/repository: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
- [results] Results, including evaluations, are reproducible via the attached R notebook: "Complete code to reproduce all of the results for this and the following section is attached as an R notebook in Additional file 1."
- [intro] ARACNE is the existing Cytoscape-plugin baseline against which Genexpi is positioned: "there are currently, only two Cytoscape plugins for gene network inference: ARACNE [8] and Network BMA [9]."
