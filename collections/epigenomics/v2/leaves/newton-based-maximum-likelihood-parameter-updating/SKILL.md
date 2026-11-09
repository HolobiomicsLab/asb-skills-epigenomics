---
name: newton-based-maximum-likelihood-parameter-updating
description: Use when fitting CscoreTool-M's probabilistic sub-compartment model (Ci,
  alpha, beta) to Hi-C interaction data and you need to reproduce or re-implement
  its non-convex likelihood optimization and convergence loop.
when_to_use_negative:
- Input is not Hi-C interaction data (e.g. ChIP-seq or RNA-seq counts); CscoreTool-M's
  likelihood model is defined over Hi-C summary interactions
- Genomic windows are not equal-length with a defined Rabl position (relative centromere-telomere
  coordinate); the Rabl-effect term of the model cannot be specified
- You only need simple compartment calls (A/B) rather than multiple probabilistic
  sub-compartments within a cell population
edam_operation: http://edamontology.org/operation_3893
edam_topics: []
tools:
- name: CscoreTool-M
  role: Implements the sequential per-chromosome Ci / alpha-beta Newton maximum-likelihood
    optimization over the Hi-C summary data
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M.cpp, twister.h and twister.cpp with -fopenmp -O3 when
    the prebuilt CentOS executable does not work
- name: HOMER
  role: Defines the accepted HiCsummary input file format produced by runHiCpca.pl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:3e5ad62ed2b2360929857ee642be96ea32c752014a2800753309ec8167341363
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - HOMER
  - g++
  edam_operation: http://edamontology.org/operation_3893
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# newton-based-maximum-likelihood-parameter-updating

## Summary

CscoreTool-M's maximum-likelihood optimization loop for Hi-C sub-compartment inference: it sequentially solves convex sub-problems for the per-window sub-compartment probabilities Ci on each chromosome using a primal-dual interior point method with Newton search as the inner solver, then updates alpha and beta by Newton updates, iterating until the likelihood converges to a local maximum.

## When to use

Use when fitting CscoreTool-M's probabilistic sub-compartment model (Ci, alpha, beta) to Hi-C interaction data and you need to reproduce or re-implement its non-convex likelihood optimization and convergence loop.

## When NOT to use

- Input is not Hi-C interaction data (e.g. ChIP-seq or RNA-seq counts); CscoreTool-M's likelihood model is defined over Hi-C summary interactions
- Genomic windows are not equal-length with a defined Rabl position (relative centromere-telomere coordinate); the Rabl-effect term of the model cannot be specified
- You only need simple compartment calls (A/B) rather than multiple probabilistic sub-compartments within a cell population

## Inputs

- Equal-length genomic window bed file with fourth column giving the Rabl position (relative position between centromere 0 and telomere 1)
- Hi-C interaction summary file in the HOMER runHiCpca.pl HiCsummary format
- N_subcompartment (number of sub-compartments, e.g. 5 or 10)
- N_Rablwindow (size of the Rabl matrix output, e.g. 20)
- N_session (number of parallel random-start sessions, e.g. 12)
- Optional ENCODE blacklist regions file (Blacklist.bed)
- Optional ExcludedInteractions.txt listing regions with translocations (chrName_chrStart_chrEnd groups per line)

## Outputs

- XXX_cscore.txt with the Cscore (sub-compartment probability) estimated for each genomic window
- XXX_cscore.bedgraph with c-scores normalized to add up to 1 per genomic region (regions with no reads omitted)
- XXX_rabl.txt with the estimated Rabl effect shown as a matrix

## How to apply

Parse the mandatory arguments (Rabl-format genomic window bed file, HOMER-format HiCsummary file, output prefix, N_subcompartment=5, N_Rablwindow=20, N_session=12) plus optional Blacklist.bed and ExcludedInteractions.txt. Initialize the probabilistic model parameters from multiple random starts, running N_session parallel sessions chosen according to available resources. Solve the per-chromosome convex sub-problems for Ci using a primal-dual interior point method whose inner solver is Newton search, then update alpha and beta by Newton updates; because each block update only increases the likelihood, iterate the cycle until convergence to a local maximum, restarting from new random initializations as needed. Before optimization, exclude interactions involving ENCODE blacklist regions (possibly affected by copy number variations) and, when ExcludedInteractions.txt is supplied, interactions among listed translocation regions, removing their expectations from the likelihood calculation rather than merely deleting the interactions.

## Related tools

- **CscoreTool-M** (Implements the sequential per-chromosome Ci / alpha-beta Newton maximum-likelihood optimization over the Hi-C summary data) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M.cpp, twister.h and twister.cpp with -fopenmp -O3 when the prebuilt CentOS executable does not work)
- **HOMER** (Defines the accepted HiCsummary input file format produced by runHiCpca.pl)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_3 5 20 12 hg38_blacklisted.bed Test_excluded.txt
```

## Evaluation signals

- All three output files (cscore.txt, cscore.bedgraph, rabl.txt) exist for the chosen output prefix
- Cscore values are interpretable as probabilities of a genomic region locating in a specific sub-compartment
- In the bedgraph output, c-scores normalize to add up to 1 for each genomic region, and regions with no reads are omitted
- The Rabl effect is emitted as a matrix of size N_Rablwindow (e.g. 20)
- The likelihood function increases monotonically with each Ci / alpha-beta block update, converging to a local maximum

## Limitations

- The optimization is non-convex; the coordinate loop converges only to a local maximum, so multiple random-start sessions (N_session) and restarts are needed
- Prebuilt executable is compiled on CentOS and may not work on other Linux systems, requiring recompilation from source
- Interactions in ENCODE blacklist regions (possibly due to copy number variations) and translocation regions must be excluded, and their expectations must also be excluded from the calculation — simply deleting interactions from the input file is not equivalent

## Evidence

- [methods] The algorithm works by sequentially optimizing Ci on each chromosome, and then optimizing alpha and beta; by iterating this procedure, the likelihood function only increases each time, converging to a local maximum.: [evidence span withheld — anchor status: not_found]
- [abstract] Here we describe a probabilistic-modeling-based method called CscoreTool-M that infers multiple 3D genome sub-compartments from Hi-C data.: "a probabilistic-modeling-based method called CscoreTool-M that infers multiple 3D genome sub-compartments from Hi-C data"
- [readme] Blacklist.bed This is the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations.: "the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations"
- [readme] The c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown.: "Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown."
- [intro] g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M: "g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M"
