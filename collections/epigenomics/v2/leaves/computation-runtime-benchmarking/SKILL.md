---
name: computation-runtime-benchmarking
description: Use when you must quantify how compute cost scales with Hi-C matrix resolution
  for a TAD-calling method, and demonstrate that chromosome-parallel execution reduces
  total runtime for a fixed cell line and parameter set.
when_to_use_negative:
- The goal is to benchmark TAD-calling accuracy (e.g., FMS or boundary anchoring ratios)
  rather than compute time — this skill measures runtime only.
- The input is a simulated or toy Hi-C matrix (e.g., Forcato et al. or Trussart et
  al. datasets) where the paper's resolution-scaling runtime claims do not apply.
- The matrix is a single small chromosome where sequential vs parallel timing differences
  are trivially small and uninformative.
tools:
- name: TOAST
  role: TAD-calling pipeline (toast.py) whose sequential and chromosome-parallel runtime
    is being measured
  repo: https://github.com/ghaiyan/TOAST
- name: PyTorch
  role: Trains the 2-layer GAE (hidden_dim 64, embedding_dim 16, Adam, lr 0.001, 200
    epochs), the dominant compute cost per submatrix
- name: HDBSCAN
  role: Unsupervised clustering of the 16-dimensional GAE embeddings, part of the
    per-chromosome pipeline being timed
- name: Juicer
  role: juicer_tools used to dump KR-normalized NxN intra-chromosomal matrices from
    the .hic file at each resolution
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:b50a8fa4900cd86299c7821ad576f701f884d5176dba120d0144554a8ea6a3a0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ghaiyan/TOAST
  repo_url: https://github.com/ghaiyan/TOAST
  tools:
  - HDBSCAN
  - Juicer
  - PyTorch
  - TOAST
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# computation-runtime-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the sequential versus parallel wall-clock runtime of the TOAST TAD-calling pipeline (GAE + HDBSCAN) on GM12878 Hi-C contact matrices at 50 Kb, 25 Kb, and 5 Kb resolutions, reproducing the reported 2 h / 4 h / 11 h sequential times and ~0.5 h parallel time.

## When to use

Use when you must quantify how compute cost scales with Hi-C matrix resolution for a TAD-calling method, and demonstrate that chromosome-parallel execution reduces total runtime for a fixed cell line and parameter set.

## When NOT to use

- The goal is to benchmark TAD-calling accuracy (e.g., FMS or boundary anchoring ratios) rather than compute time — this skill measures runtime only.
- The input is a simulated or toy Hi-C matrix (e.g., Forcato et al. or Trussart et al. datasets) where the paper's resolution-scaling runtime claims do not apply.
- The matrix is a single small chromosome where sequential vs parallel timing differences are trivially small and uninformative.

## Inputs

- GM12878 .hic file (accession 4DNFI1UEG1HD, GEO GSE63525)
- KR-normalized NxN intra-chromosomal Hi-C contact matrices at 50 Kb, 25 Kb, and 5 Kb resolutions, chromosomes 1–22
- TOAST pipeline scripts (toast.py) from https://github.com/ghaiyan/TOAST

## Outputs

- runtime_report.csv with per-resolution sequential wall-clock times and parallel elapsed time, annotated with reference runtimes
- TAD calls per chromosome/resolution (merge-file output of toast.py) used to confirm the parallel run produced equivalent results

## How to apply

Obtain the GM12878 .hic file (4DNFI1UEG1HD, GEO GSE63525) and dump KR-normalized NxN intra-chromosomal matrices at 50 Kb, 25 Kb, and 5 Kb with Juicer tools. Run toast.py sequentially on all chromosomes (1–22) per resolution using the paper configuration: non-overlapping 400x400 submatrices, 2-layer GAE with hidden dimension 64 and embedding dimension 16, learning rate 0.001, 200 epochs with Adam, HDBSCAN clustering of the 16-dim embeddings, and a 200 Kb minimum contiguous-region TAD length; record per-resolution wall-clock time. Re-run in parallel mode across chromosomes and record total elapsed time. Aggregate sequential and parallel timings into runtime_report.csv, annotating each measurement with the reported reference runtime for direct comparison.

## Related tools

- **TOAST** (TAD-calling pipeline (toast.py) whose sequential and chromosome-parallel runtime is being measured) — https://github.com/ghaiyan/TOAST
- **PyTorch** (Trains the 2-layer GAE (hidden_dim 64, embedding_dim 16, Adam, lr 0.001, 200 epochs), the dominant compute cost per submatrix)
- **HDBSCAN** (Unsupervised clustering of the 16-dimensional GAE embeddings, part of the per-chromosome pipeline being timed)
- **Juicer** (juicer_tools used to dump KR-normalized NxN intra-chromosomal matrices from the .hic file at each resolution)

## Examples

```
cd /src && python toast.py  # set input GM12878 NxN matrix folder (50/25/5 Kb) and output folder per run, timing each resolution
```

## Evaluation signals

- Measured sequential runtimes at 50 Kb, 25 Kb, and 5 Kb are consistent with the reported approximate values of 2 h, 4 h, and 11 h.
- The chromosome-parallel run over all GM12878 TAD calls completes in roughly 0.5 h.
- runtime_report.csv contains one annotated row per resolution (sequential) plus the parallel measurement, each paired with its reported reference runtime.
- TAD outputs from the parallel run match those from the sequential run, confirming parallelization did not alter results.
- Runtime increases monotonically with resolution (50 Kb < 25 Kb < 5 Kb), consistent with matrix-size scaling.

## Limitations

- Reference times (2 h / 4 h / 11 h sequential, ~0.5 h parallel) are hardware-dependent; reproductions on other machines should be compared relatively, not absolutely.
- The 200 Kb minimum TAD length and 400x400 submatrix partitioning are fixed in this benchmark; changing them changes runtime and invalidates direct comparison.
- Timing covers the whole TOAST pipeline (data preprocessing, GAE training, HDBSCAN clustering, TAD generation), so per-module costs are not separable without extra instrumentation.

## Evidence

- [results] Sequential scaling across resolutions: "we experiment sequentially on the 50 Kb, 25 Kb, and 5 Kb Hi-C contact matrices of the GM12878 cell line for all chromosomes (from 1 to 22)"
- [methods] Pipeline modules being timed: "A data processing module, a GAE module, a clustering module, and a TAD generation module"
- [methods] TAD length filter held constant: "If the length of a contiguous region exceeds 200 Kb, it can be identified as a TAD"
- [readme] Matrix preparation step: "For .hic format data, you can dump data using juicer_tools.js and get the NxN matrix"
