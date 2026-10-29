---
name: parallel-versus-sequential-execution-timing
description: 'Use when reproducing or benchmarking compute-time claims for a TAD-calling
  or similar pipeline: input is Hi-C contact matrices at multiple resolutions (50
  Kb, 25 Kb, 5 Kb) and the goal is to measure and compare sequential vs parallel wall-clock
  runtime.'
when_to_use_negative:
- The goal is to compare TAD accuracy (FMS, boundary anchors) rather than execution
  time — use evaluation-oriented TAD comparison workflows instead.
- Input data are simulated Hi-C contact matrices from Forcato et al. or Trussart et
  al. used for noise-robustness validation, where runtime scaling on real multi-resolution
  GM12878 matrices is not the question.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3448
tools:
- name: TOAST
  role: TAD-calling pipeline (toast.py) whose sequential and parallel executions are
    timed
  repo: https://github.com/ghaiyan/TOAST
- name: Juicer
  role: juicer_tools used to dump KR-normalized NxN intra-chromosomal Hi-C contact
    matrices from .hic files
- name: PyTorch
  role: framework for the 2-layer GAE training inside each timed TOAST run
- name: HDBSCAN
  role: unsupervised clustering of the 16-dimensional GAE embeddings within the timed
    pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.09.019
    title: 'TOAST: A novel method for identifying topologically associated domains
      based on graph auto-encoders and clustering'
  source_content_hash: sha256:81629c1c7ba202c01e47dd221a7e94875149e883d3978e82c7c2eec45d7741ad
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3448
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# parallel-versus-sequential-execution-timing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks the wall-clock runtime of a computational pipeline (TOAST TAD calling on GM12878 Hi-C matrices) when run sequentially per resolution versus in parallel across chromosomes, to quantify speedups achievable through parallel computing.

## When to use

Use when reproducing or benchmarking compute-time claims for a TAD-calling or similar pipeline: input is Hi-C contact matrices at multiple resolutions (50 Kb, 25 Kb, 5 Kb) and the goal is to measure and compare sequential vs parallel wall-clock runtime.

## When NOT to use

- The goal is to compare TAD accuracy (FMS, boundary anchors) rather than execution time — use evaluation-oriented TAD comparison workflows instead.
- Input data are simulated Hi-C contact matrices from Forcato et al. or Trussart et al. used for noise-robustness validation, where runtime scaling on real multi-resolution GM12878 matrices is not the question.

## Inputs

- GM12878 .hic file (4DNFI1UEG1HD, GEO GSE63525)
- KR-normalized NxN intra-chromosomal Hi-C contact matrices at 50 Kb, 25 Kb, and 5 Kb resolutions, dumped with Juicer tools

## Outputs

- Per-resolution wall-clock times for sequential TOAST runs (expected ~2 h at 50 Kb, ~4 h at 25 Kb, ~11 h at 5 Kb)
- Total elapsed time for the parallel run across chromosomes (expected ~0.5 h)
- runtime_report.csv aggregating measured and reference runtimes
- TAD calls per chromosome at each resolution

## How to apply

Obtain the GM12878 .hic file (accession 4DNFI1UEG1HD, GEO GSE63525) and use Juicer tools to dump KR-normalized NxN intra-chromosomal contact matrices at 50 Kb, 25 Kb, and 5 Kb. Run toast.py sequentially for all chromosomes at each resolution using the paper's configuration: non-overlapping 400x400 submatrices, a 2-layer GAE with hidden dimension 64 and embedding dimension 16, learning rate 0.001, 200 epochs with Adam, HDBSCAN clustering, and a 200 Kb minimum TAD length, recording per-resolution wall-clock time. Then run TOAST in parallel mode across chromosomes and record the total elapsed time. The rationale is that genome-scale TAD calling is embarrassingly parallel across chromosomes, so parallelization collapses many hours of sequential compute into under an hour; aggregate measured and reference runtimes in runtime_report.csv for direct comparison.

## Related tools

- **TOAST** (TAD-calling pipeline (toast.py) whose sequential and parallel executions are timed) — https://github.com/ghaiyan/TOAST
- **Juicer** (juicer_tools used to dump KR-normalized NxN intra-chromosomal Hi-C contact matrices from .hic files)
- **PyTorch** (framework for the 2-layer GAE training inside each timed TOAST run)
- **HDBSCAN** (unsupervised clustering of the 16-dimensional GAE embeddings within the timed pipeline)

## Examples

```
juicer_tools.sh dump observed/KR GM12878.hic 5 KB chr1 NxN.txt; python toast.py --input NxN.txt --output tads_chr1.txt
```

## Evaluation signals

- Sequential runtimes should be consistent with reported approximate values: ~2 h (50 Kb), ~4 h (25 Kb), ~11 h (5 Kb).
- Parallel run across chromosomes should complete in roughly 0.5 h total.
- Runtime should scale with matrix size/resolution: finer 5 Kb matrices take far longer than 50 Kb ones.
- runtime_report.csv should contain all six measurements (three sequential resolutions plus parallel) annotated with their reported reference runtimes.
- TAD calls produced under parallel mode should cover all chromosomes 1–22, matching the sequential output set.

## Limitations

- Sequential runtime scales strongly with resolution — the 5 Kb matrix takes about 11 hours versus 2 hours at 50 Kb — so reproducing full sequential benchmarks is expensive without parallelization.
- Timing measurements depend on hardware and the fixed GAE configuration (200 epochs, Adam, lr 0.001); they are not transferable across different network settings or machine loads.

## Evidence
