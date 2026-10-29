---
name: hic-subcompartment-probabilistic-inference
description: Use when Hi-C interaction summary data (HOMER HiCsummary format) must
  be segmented into more than the canonical A/B compartments — e.g. inferring 5–10
  sub-compartments, quantifying cell-population heterogeneity, or deconvolving mixed
  cell-type samples into cell-type-specific profiles.
when_to_use_negative:
- Input is not raw Hi-C interaction summary data but already-computed compartment
  calls or feature tables — CscoreTool-M requires a HOMER-format HiCsummary of individual
  interactions
- Only a binary A/B compartment assignment is needed; simpler PCA-based approaches
  (e.g. HOMER runHiCpca.pl) suffice for two compartments
- The sample is a single homogeneous cell type where population heterogeneity deconvolution
  is not the goal — the multi-sub-compartment model adds complexity without benefit
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CscoreTool-M
  role: 'Primary executable: probabilistic inference of multiple 3D genome sub-compartments
    from Hi-C summary data, producing per-window Cscores'
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiler used to build CscoreTool-M from source (CscoreTool-M.cpp, twister.h,
    twister.cpp) with -fopenmp -O3 if the precompiled Linux binary fails
- name: HOMER
  role: Defines the accepted HiCsummary input file format via runHiCpca.pl; CscoreTool-M
    accepts the same format
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:7af9227da1c9294691140706d1151e9d931f74421d0e8858d9cbd9e7ffd810f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - HOMER
  - g++
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# hic-subcompartment-probabilistic-inference

## Summary

Probabilistic-modeling-based inference of multiple 3D genome sub-compartments from Hi-C data using CscoreTool-M, which outputs per-window compartment scores (Cscores) interpretable as probabilities of a region belonging to a specific sub-compartment. Enables deconvolution of mixed cell populations into cell-type-specific sub-compartments.

## When to use

Use when Hi-C interaction summary data (HOMER HiCsummary format) must be segmented into more than the canonical A/B compartments — e.g. inferring 5–10 sub-compartments, quantifying cell-population heterogeneity, or deconvolving mixed cell-type samples into cell-type-specific profiles.

## When NOT to use

- Input is not raw Hi-C interaction summary data but already-computed compartment calls or feature tables — CscoreTool-M requires a HOMER-format HiCsummary of individual interactions
- Only a binary A/B compartment assignment is needed; simpler PCA-based approaches (e.g. HOMER runHiCpca.pl) suffice for two compartments
- The sample is a single homogeneous cell type where population heterogeneity deconvolution is not the goal — the multi-sub-compartment model adds complexity without benefit

## Inputs

- Equal-length genomic windows bed file with Rabl position (relative centromere-telomere position) in the 4th column
- Hi-C interaction summary file in HOMER runHiCpca.pl HiCsummary format
- Optional: ENCODE blacklist bed file (Blacklist.bed) for copy-number-variation-affected regions
- Optional: ExcludedInteractions.txt listing translocation regions in chrName_chrStart_chrEnd format

## Outputs

- <prefix>_cscore.txt — Cscore (sub-compartment probability) per genomic window
- <prefix>_cscore.bedgraph — normalized c-scores (summing to 1 per region) for visualization
- <prefix>_rabl.txt — estimated Rabl effect as a matrix

## How to apply

Prepare an equal-length genomic windows bed file whose fourth column gives the Rabl position (relative position between centromere 0 and telomere 1), and a Hi-C summary file in the same format as HOMER runHiCpca.pl's HiCsummary. Optionally supply an ENCODE Blacklist.bed (7th argument) to exclude regions affected by copy-number variation and an ExcludedInteractions.txt file (8th argument) listing tab-separated chrName_chrStart_chrEnd regions whose interactions (including in expectation calculation) should be excluded due to translocations. Run './CscoreTool-M <Rabl.bed> <summary> <prefix> <N_subcompartments> <N_Rablwindow> <N_session>', e.g. 10 sub-compartments with Rabl matrix size 20 and 12 parallel sessions for the mixed GM12878+IMR90 simulation. Parse <prefix>_cscore.txt for per-window Cscores; for mixed-population deconvolution, correlate each inferred sub-compartment score profile against single-cell-type-derived model parameters and confirm they separate into cell-type-specific groups (e.g. five GM12878-like Sim1–Sim5 and five IMR90-like Sim6–Sim10).

## Related tools

- **CscoreTool-M** (Primary executable: probabilistic inference of multiple 3D genome sub-compartments from Hi-C summary data, producing per-window Cscores) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiler used to build CscoreTool-M from source (CscoreTool-M.cpp, twister.h, twister.cpp) with -fopenmp -O3 if the precompiled Linux binary fails)
- **HOMER** (Defines the accepted HiCsummary input file format via runHiCpca.pl; CscoreTool-M accepts the same format)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12
```

## Evaluation signals

- All three output files (<prefix>_cscore.txt, <prefix>_cscore.bedgraph, <prefix>_rabl.txt) are produced with the expected per-window and matrix structures
- In the bedgraph output, normalized c-scores sum to 1 for each genomic region, and regions with no reads are omitted
- For mixed-population deconvolution (e.g. GM12878 + IMR90 simulated mix, N_subcompartments=10), correlation of inferred sub-compartment profiles against the single-cell-type model parameters separates them into the expected cell-type-specific groups (five GM12878-like and five IMR90-like)
- Running with the 7th/8th optional arguments correctly excludes blacklisted and translocation regions, with expectations also excluded for ExcludedInteractions
- Sub-compartment assignments show improved accuracy over published methods for both active and repressed chromatin, per the paper's benchmark

## Limitations

- The precompiled executable was built on CentOS Linux; on other systems it may not run and must be recompiled from CscoreTool-M.cpp, twister.h, and twister.cpp
- Blacklist exclusion is recommended because blacklisted regions may be affected by copy number variations; translocation-containing regions require explicit ExcludedInteractions.txt since simple deletion of interactions from the input is insufficient (expectations must also be recalculated)
- Rabl position must be provided per window in the bed file; the method's model assumes a Rabl-like organization parameterized by N_Rablwindow
- Number of sessions (N_session) is a parallelization parameter whose choice depends on available resources, not a statistical parameter

## Evidence

- [abstract] Here we describe a probabilistic-modeling-based method called CscoreTool-M that infers multiple 3D genome sub-compartments from Hi-C data.: "Here we describe a probabilistic-modeling-based method called CscoreTool-M that infers multiple 3D genome sub-compartments from Hi-C data."
- [results] five of the 10 inferred sub-compartments resembled the GM12878 model: [evidence span withheld — anchor status: not_found]
- [readme] CscoreTool-M command-line usage from README: "Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]"
- [readme] Rabl position definition in windows.bed: "It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1)."
- [readme] ExcludedInteractions expectation handling: "Note that the expectation are also excluded in the calculation, so it's different from simply delete the interactions from the input file."
- [intro] HOMER HiCsummary format compatibility: "We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl."
