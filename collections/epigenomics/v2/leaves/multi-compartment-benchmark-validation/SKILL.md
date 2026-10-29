---
name: multi-compartment-benchmark-validation
description: Use when evaluating whether a Hi-C deconvolution tool can resolve multiple
  sub-compartments, or when validating that sub-compartments inferred from a mixed
  cell population correspond to the underlying cell-type-specific models.
when_to_use_negative:
- The goal is single A/B compartment calling (e.g. HOMER runHiCpca.pl eigenvector
  analysis) rather than probabilistic multi-sub-compartment inference
- The input is not Hi-C interaction data in HiCsummary format — e.g. already-aggregated
  contact matrices or ChIP-seq signal tracks
- Interpretation of inferred sub-compartments as discrete binary compartments is required;
  CscoreTool-M scores are probabilities of sub-compartment localization within a cell
  population
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3922
- http://edamontology.org/topic_0602
tools:
- name: CscoreTool-M
  role: Probabilistic-modeling tool that infers multiple 3D genome sub-compartments
    and per-window compartment scores from Hi-C summary data
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M.cpp, twister.h and twister.cpp with -fopenmp -O3 if
    the prebuilt Linux executable does not work
- name: HOMER
  role: Defines the accepted HiCsummary input file format (runHiCpca.pl format) used
    by CscoreTool-M
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:fae02ce02c3c49d9c9760ceab5423f71534a5e2ea9f9c718048dca4b71792a93
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
  - http://edamontology.org/topic_3922
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-compartment-benchmark-validation

## Summary

Benchmark and validate CscoreTool-M's probabilistic inference of multiple 3D genome sub-compartments from Hi-C data, including mixed-population deconvolution of simulated cell-type mixtures (e.g. GM12878 + IMR90) into cell-type-specific sub-compartments.

## When to use

Use when evaluating whether a Hi-C deconvolution tool can resolve multiple sub-compartments, or when validating that sub-compartments inferred from a mixed cell population correspond to the underlying cell-type-specific models.

## When NOT to use

- The goal is single A/B compartment calling (e.g. HOMER runHiCpca.pl eigenvector analysis) rather than probabilistic multi-sub-compartment inference
- The input is not Hi-C interaction data in HiCsummary format — e.g. already-aggregated contact matrices or ChIP-seq signal tracks
- Interpretation of inferred sub-compartments as discrete binary compartments is required; CscoreTool-M scores are probabilities of sub-compartment localization within a cell population

## Inputs

- Hi-C summary file (HOMER runHiCpca.pl HiCsummary format) for each cell-type model, depth-matched and merged for the mixed population
- Equal-length genomic windows bed file with fourth column encoding Rabl position (relative position between centromere (0) and telomere (1))
- Command-line parameters: N_subcompartments (e.g. 10), N_Rablwindow (e.g. 20), N_session (e.g. 12)
- Optional: ENCODE blacklist bed file (Blacklist.bed) and ExcludedInteractions.txt for translocation exclusion

## Outputs

- <prefix>_cscore.txt — Cscore estimated for each genomic window (probability of locating in a specific sub-compartment)
- <prefix>_cscore.bedgraph — visualization file with c-scores normalized to add up to 1 per genomic region
- <prefix>_rabl.txt — estimated Rabl effect as a matrix
- Per-sub-compartment correlation profiles against cell-type model parameters for validation

## How to apply

Simulate two independent Hi-C summary datasets parameterized from the two cell-type models of interest (e.g. GM12878 and IMR90), depth-match them before merging into a single mixed-population summary file in HOMER HiCsummary format. Run CscoreTool-M with an equal-length Rabl bed file (fourth column giving relative position between centromere (0) and telomere (1)), specifying N_subcompartment=10, genomic window size=5, N_Rablwindow=20, and N_session=12 parallel sessions. Parse the output <prefix>_cscore.txt containing the Cscore estimated per genomic window, plus the normalized <prefix>_cscore.bedgraph and the Rabl effect matrix <prefix>_rabl.txt. Correlate each of the 10 inferred sub-compartment score profiles against the GM12878- and IMR90-derived model parameters used in the simulation. Validation succeeds when the 10 inferred sub-compartments separate cleanly — e.g. exactly five GM12878-like profiles (Sim1-Sim5) and five IMR90-like profiles (Sim6-Sim10) by correlation with the model parameters.

## Related tools

- **CscoreTool-M** (Probabilistic-modeling tool that infers multiple 3D genome sub-compartments and per-window compartment scores from Hi-C summary data) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M.cpp, twister.h and twister.cpp with -fopenmp -O3 if the prebuilt Linux executable does not work)
- **HOMER** (Defines the accepted HiCsummary input file format (runHiCpca.pl format) used by CscoreTool-M)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12
```

## Evaluation signals

- In the mixed GM12878 + IMR90 simulation with 10 sub-compartments, five inferred sub-compartments (Sim1-Sim5) correlate with the GM12878 model and the other five (Sim6-Sim10) with the IMR90 model
- Cscores in <prefix>_cscore.bedgraph are normalized to add up to 1 for each genomic region, with regions having no reads omitted
- Three output files exist per run: _cscore.txt (per-window Cscore), _cscore.bedgraph, and _rabl.txt (Rabl effect matrix)
- Compartment scores behave as probabilities of a genomic region locating in a specific sub-compartment
- Runs with 7 or 8 arguments correctly interpret argument 7 as Blacklist.bed and argument 8 as ExcludedInteractions.txt

## Limitations

- The CscoreTool-M executable is compiled on CentOS Linux and may not run on other systems; recompilation from source with g++ -fopenmp -O3 is needed
- Optional filter files change expectations: the 7th argument is taken as Blacklist.bed (ENCODE blacklist regions excluded possibly due to copy number variations) and the 8th as ExcludedInteractions.txt for translocation exclusion, where expectations are also excluded from calculation — not equivalent to deleting interactions from the input file
- Inferred sub-compartments are probabilistic within a cell population, and heterogeneity is higher in proliferating cells (likely due to cell-cycle stage differences), complicating interpretation in mixed populations

## Evidence

- [abstract] CscoreTool-M infers sub-compartments probabilistically: "Here we describe a probabilistic-modeling-based method called CscoreTool-M that infers multiple 3D genome sub-compartments from Hi-C data."
- [abstract] Cscores represent sub-compartment localization probability: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment."
- [abstract] CscoreTool-M outperforms published methods: "Compared to published methods, CscoreTool-M is more accurate in inferring sub-compartments corresponding to both active and repressed chromatin."
- [methods] Methods invocation example: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12"
- [readme] Rabl bed file specification: "It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1)."
- [readme] Output normalization semantics: "Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown."
- [readme] HiCsummary input format: "We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl."
