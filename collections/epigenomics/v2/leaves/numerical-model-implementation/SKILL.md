---
name: numerical-model-implementation
description: Use when implementing or re-deriving a probabilistic model of Hi-C interaction
  counts that requires a centromere-telomere (Rabl) positional term, bias factors,
  and sub-compartment probability products feeding a Poisson likelihood.
when_to_use_negative:
- The analysis targets cis interactions or single-cell Hi-C rather than population-level
  trans-interaction counts modeled by the Rabl term
- Observed reads fall in ENCODE blacklisted regions or span regions with translocations
  listed in ExcludedInteractions.txt, which the model excludes from both counts and
  expectations
- Only a two-compartment (A/B) assignment is needed and no sub-compartment probability
  model is required
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3420
tools:
- name: CscoreTool-M
  role: Executes the Rabl-effect/Poisson generative model to infer sub-compartment
    probabilities and output the estimated Rabl effect matrix
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M.cpp with twister.cpp using -fopenmp -O3 when the prebuilt
    single-file Linux executable does not run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:40a475298fae39b157d714b60d4ad60b5d0f9a96df6aeaf42faf1d897a6485ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3420
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# numerical-model-implementation

## Summary

Implements the CscoreTool-M generative model for Hi-C trans interactions: a Rabl-effect modifier Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj combined with window biases Bi/Bj and sub-compartment probabilities to yield expected counts Fij evaluated under a Poisson log-likelihood.

## When to use

Use when implementing or re-deriving a probabilistic model of Hi-C interaction counts that requires a centromere-telomere (Rabl) positional term, bias factors, and sub-compartment probability products feeding a Poisson likelihood.

## When NOT to use

- The analysis targets cis interactions or single-cell Hi-C rather than population-level trans-interaction counts modeled by the Rabl term
- Observed reads fall in ENCODE blacklisted regions or span regions with translocations listed in ExcludedInteractions.txt, which the model excludes from both counts and expectations
- Only a two-compartment (A/B) assignment is needed and no sub-compartment probability model is required

## Inputs

- Equal-length genomic windows bed file with Rabl position in column 4 (relative coordinate 0=centromere to 1=telomere, e.g. hg38_500k_Rabl.bed)
- Hi-C interaction summary file in HOMER runHiCpca.pl HiCsummary format (e.g. test.summary from 0.5% randomly selected chr1 reads of the high-resolution GM12878 Hi-C dataset)
- Fitted Rabl coefficients alpha and beta
- Per-window bias factors Bi and sub-compartment probability vectors Pik
- Optional ENCODE Blacklist.bed regions
- Optional ExcludedInteractions.txt for regions with translocations

## Outputs

- Rij / expected interaction count matrix Fij (XXX_rabl.txt, estimated Rabl effect as a matrix)
- Poisson log-likelihood value per Equation (4) as a finite scalar
- Per-window Cscore file (XXX_cscore.txt) with scores normalized to sum to 1 per genomic region in the bedgraph visualization output

## How to apply

Load an equal-length genomic window bed file (e.g. hg38_500k_Rabl.bed) whose fourth column gives the Rabl position, the relative coordinate between centromere (0) and telomere (1), and parse the CscoreTool-M arguments (N_subcompartments, N_Rablwindow, N_session). For every window pair (i, j), compute the Rabl modifier Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj from their Rabl coordinates ri, rj and the fitted alpha and beta coefficients. Form the expected interaction count Fij = Bi*Bj*Rij*Σk(Pik*Pjk), combining window biases with the sum over sub-compartments of the products of the sub-compartment probability vectors Pik and Pjk. Evaluate the Poisson log-likelihood of Equation (4) using observed trans Hi-C reads Nij against Fij, compiling the C++ source with 'g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3' and running with N_session tuned to available compute resources. Write the estimated Rabl effect as a matrix (XXX_rabl.txt) bounded by N_Rablwindow, excluding blacklisted regions and translocated region pairs when the optional 7th and 8th arguments are supplied, since excluded interactions remove the expectation too — not just the observed reads.

## Related tools

- **CscoreTool-M** (Executes the Rabl-effect/Poisson generative model to infer sub-compartment probabilities and output the estimated Rabl effect matrix) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M.cpp with twister.cpp using -fopenmp -O3 when the prebuilt single-file Linux executable does not run)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12
```

## Evaluation signals

- Output matrix file exists with dimensions bounded by N_Rablwindow
- Reported Poisson log-likelihood is a finite scalar
- XXX_cscore.txt Cscore values, when converted to the bedgraph visualization, are normalized to add up to 1 for each genomic region
- Regions with no reads are absent from the bedgraph output
- Interactions involving blacklisted or translocation-excluded regions are absent from both observed counts and expectation calculations

## Limitations

- The prebuilt executable is compiled on a CentOS Linux machine and may not run elsewhere, requiring recompilation from CscoreTool-M.cpp, twister.h and twister.cpp
- N_session must be chosen according to available compute resources; too small a value may stall the analysis on limited hardware
- Blacklisted regions (possible copy number variations) and translocated region pairs must be excluded explicitly, since the tool excludes their expectation as well, not just the observed interactions

## Evidence

- [readme] Rabl term and Poisson likelihood: "XXX_rabl.txt estimated Rabl effect, shown as a matrix."
- [readme] Output normalization: "XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown."
- [readme] Input bed format: "It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1)."
- [readme] ExcludedInteractions semantics: "Note that the expectation are also excluded in the calculation, so it's different from simply delete the interactions from the input file."
- [intro] Compilation: "g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M"
