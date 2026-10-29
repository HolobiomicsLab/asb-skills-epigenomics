---
name: hi-c-interaction-modeling
description: Use when fitting a generative model of Hi-C interaction counts (e.g.
  CscoreTool-M) where expected counts Fij must incorporate Rabl-position effects,
  bias factors, and per-window sub-compartment probabilities, and the likelihood of
  observed trans reads is needed.
when_to_use_negative:
- The input interactions involve regions with known translocations but no ExcludedInteractions.txt
  is supplied — the README notes excluded expectations differ from simply deleting
  interactions, so omitting this file biases the model differently than intended
- Interactions within blacklisted regions (e.g. encode blacklist regions with copy
  number variations) that should be excluded via Blacklist.bed rather than modeled
- The goal is a visualization-only compartment track rather than probabilistic modeling
  — the bedgraph output (XXX_cscore.bedgraph) normalizes c-scores to add up to 1 per
  region and drops regions with no reads, which is not the Poisson model itself
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0622
tools:
- name: CscoreTool-M
  role: Probabilistic-modeling program that computes the Rabl-effect term Rij, expected
    counts Fij = Bi*Bj*Rij*Σk(Pik*Pjk), and the Poisson log-likelihood (Equation 4)
    for Hi-C trans-interaction fragments
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiler used to build CscoreTool-M from CscoreTool-M.cpp and twister.cpp
    with -fopenmp -O3 when the prebuilt Linux executable does not work
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:5745da71282da7355cba0502e4f6222f007ce7e7a9786e7f2c4178f9a54a460a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hi-c-interaction-modeling

## Summary

Model expected Hi-C trans-interaction counts between genomic windows with a Rabl-effect term Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj combined with window biases Bi, Bj and sub-compartment probability products, then evaluate the Poisson log-likelihood of observed reads as in CscoreTool-M Section 2.1.1.

## When to use

Use when fitting a generative model of Hi-C interaction counts (e.g. CscoreTool-M) where expected counts Fij must incorporate Rabl-position effects, bias factors, and per-window sub-compartment probabilities, and the likelihood of observed trans reads is needed.

## When NOT to use

- The input interactions involve regions with known translocations but no ExcludedInteractions.txt is supplied — the README notes excluded expectations differ from simply deleting interactions, so omitting this file biases the model differently than intended
- Interactions within blacklisted regions (e.g. encode blacklist regions with copy number variations) that should be excluded via Blacklist.bed rather than modeled
- The goal is a visualization-only compartment track rather than probabilistic modeling — the bedgraph output (XXX_cscore.bedgraph) normalizes c-scores to add up to 1 per region and drops regions with no reads, which is not the Poisson model itself

## Inputs

- Equal-length genomic window bed file with Rabl position in the fourth column (e.g. hg38_500k_Rabl.bed), where Rabl position is the relative coordinate between centromere (0) and telomere (1)
- Hi-C interaction summary file in HOMER runHiCpca.pl HiCsummary format (e.g. Test.summary)
- Fitted alpha and beta parameters for the Rabl-effect term
- Window bias factors Bi, Bj and per-window sub-compartment probabilities Pik
- Optional Blacklist.bed (encode blacklist regions) and ExcludedInteractions.txt (regions with translocations)

## Outputs

- Rabl-effect matrix Rij / expected interaction counts Fij written as XXX_rabl.txt, sized by N_Rablwindow
- Poisson log-likelihood value from Equation (4) (finite scalar)
- XXX_cscore.txt with Cscore estimated for each genomic window

## How to apply

Load the equal-length genomic window bed file (e.g. hg38_500k_Rabl.bed) whose fourth column gives the Rabl position, the relative coordinate between centromere (0) and telomere (1), and parse the CscoreTool-M parameters N_subcompartment, N_Rablwindow, and N_session. For every pair of windows i and j compute the Rabl-effect modifier Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj using their Rabl coordinates ri, rj and the fitted alpha and beta. Form the expected interaction count Fij = Bi*Bj*Rij*Σk(Pik*Pjk), combining window biases with the sum over sub-compartments of the product of sub-compartment probabilities. Model the observed Hi-C interaction reads Nij between loci i and j as Poisson with parameter Fij and evaluate the log-likelihood of Equation (4); CscoreTool-M is compiled with g++ -fopenmp -O3 from CscoreTool-M.cpp, twister.h and twister.cpp. The estimated Rabl effect is output as a matrix in XXX_rabl.txt, sized by N_Rablwindow (e.g. 20 in the example run).

## Related tools

- **CscoreTool-M** (Probabilistic-modeling program that computes the Rabl-effect term Rij, expected counts Fij = Bi*Bj*Rij*Σk(Pik*Pjk), and the Poisson log-likelihood (Equation 4) for Hi-C trans-interaction fragments) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiler used to build CscoreTool-M from CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 when the prebuilt Linux executable does not work)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12
```

## Evaluation signals

- XXX_rabl.txt exists and its matrix dimensions match N_Rablwindow (e.g. 20 in the example invocation)
- The reported Poisson log-likelihood of Equation (4) is a finite scalar
- Each row of XXX_cscore.txt c-scores, when normalized for the bedgraph output, adds up to 1 for each genomic region
- Interaction pairs overlapping entries in Blacklist.bed or ExcludedInteractions.txt are excluded from both observed counts and expectations
- Rij values follow the parametric form 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj given the Rabl positions in the bed fourth column

## Limitations

- The prebuilt executable is compiled on a CentOS linux machine and may not work elsewhere; recompilation with g++ -fopenmp -O3 from CscoreTool-M.cpp, twister.h and twister.cpp is required in that case
- Regions with no reads are not shown in the bedgraph output, so coverage gaps affect the visualization even though the model still fits them
- Excluding interactions via ExcludedInteractions.txt also excludes their expectations from the calculation, which is different from simply deleting interactions from the input file
- The N_session parameter choice depends on the computational resources available

## Evidence

- [methods] In the CscoreTool-M model of Section 2.1.1, expected trans-interaction fragments are Fij = Bi*Bj*Rij*Σk(Pik*Pjk): [evidence span withheld — anchor status: not_found]
- [methods] observed Hi-C interaction reads modeled as Poisson: [evidence span withheld — anchor status: not_found]
- [readme] Rabl position definition in bed file: "It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1)."
- [methods] Rabl effect output as matrix: "XXX_rabl.txt estimated Rabl effect, shown as a matrix."
- [intro] compile command with OpenMP: "g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M"
