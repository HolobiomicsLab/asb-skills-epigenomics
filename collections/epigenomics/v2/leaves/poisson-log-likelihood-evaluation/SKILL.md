---
name: poisson-log-likelihood-evaluation
description: Use when modeling trans Hi-C interactions between equal-length genomic
  windows annotated with Rabl positions (relative centromere-telomere coordinates),
  or when fitting/evaluating the CscoreTool-M generative model with fitted alpha and
  beta parameters.
when_to_use_negative:
- The analysis goal is cis-interaction compartment calling without a Rabl chromosome-organization
  model — the Rij term is specific to trans/Rabl modeling
- Observed Hi-C counts are already aggregated per pair and cannot be paired with window
  Rabl coordinates, so ri, rj and the summary-file reads cannot be joined
- Only visualization of compartment scores is needed (use XXX_cscore.bedgraph output
  instead of re-deriving the likelihood)
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0102
tools:
- name: CscoreTool-M
  role: Computes the Rij/Fij matrices and evaluates the Poisson log-likelihood over
    the observed Hi-C trans reads
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 to build the executable
    that evaluates the likelihood
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:08cb29617ac967a7acbf6e5502b5b3b55a5dd69f429cbc2d71099b5cc4f091eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# poisson-log-likelihood-evaluation

## Summary

Computes the Rabl-effect modifier Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj, combines it with bias factors Bi, Bj and sub-compartment probabilities into expected trans-interaction counts Fij = Bi*Bj*Rij*Σk(Pik*Pjk), and evaluates the Poisson log-likelihood (Equation 4 of CscoreTool-M) against observed Hi-C trans reads.

## When to use

Use when modeling trans Hi-C interactions between equal-length genomic windows annotated with Rabl positions (relative centromere-telomere coordinates), or when fitting/evaluating the CscoreTool-M generative model with fitted alpha and beta parameters.

## When NOT to use

- The analysis goal is cis-interaction compartment calling without a Rabl chromosome-organization model — the Rij term is specific to trans/Rabl modeling
- Observed Hi-C counts are already aggregated per pair and cannot be paired with window Rabl coordinates, so ri, rj and the summary-file reads cannot be joined
- Only visualization of compartment scores is needed (use XXX_cscore.bedgraph output instead of re-deriving the likelihood)

## Inputs

- Rabl-annotated equal-length genomic windows bed file (fourth column = relative centromere-telomere position, e.g. hg38_500k_Rabl.bed)
- Hi-C interaction summary file in HOMER runHiCpca.pl HiCsummary format (e.g. Test.summary)
- Fitted Rabl effect parameters alpha and beta
- Sub-compartment probability matrix P and window bias factors Bi
- Optional ENCODE blacklist regions bed file (Blacklist.bed)
- Optional translocation exclusion file (ExcludedInteractions.txt)

## Outputs

- Estimated Rabl effect matrix (XXX_rabl.txt) with dimensions bounded by N_Rablwindow
- Poisson log-likelihood value (scalar) of Equation (4)
- Expected interaction count matrix Fij for window pairs

## How to apply

Parse the Rabl bed file (e.g. hg38_500k_Rabl.bed), whose fourth column gives the relative position between centromere (0) and telomere (1), and read the fitted alpha and beta parameters. For every pair of genomic windows i and j, compute the Rabl modifier Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj from their Rabl coordinates ri and rj. Convert the Rij matrix into expected interaction counts Fij = Bi*Bj*Rij*Σk(Pik*Pjk), combining bias factors with the sub-compartment probability matrix P. Evaluate the Poisson log-likelihood of Equation (4) using the observed interaction reads Nij from the input.summary file (HOMER runHiCpca.pl HiCsummary format) with Fij as the Poisson parameter. Write the Rij/Fij matrix to the rabl output file, with dimensions bounded by N_Rablwindow. Optionally exclude blacklisted regions (ENCODE blacklist bed) and translocation interactions before evaluating the likelihood.

## Related tools

- **CscoreTool-M** (Computes the Rij/Fij matrices and evaluates the Poisson log-likelihood over the observed Hi-C trans reads) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M.cpp and twister.cpp with -fopenmp -O3 to build the executable that evaluates the likelihood)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12
```

## Evaluation signals

- Output rabl matrix file exists with dimensions matching N_Rablwindow
- Reported Poisson log-likelihood is a finite scalar
- Rij values follow the parametric form Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj for the supplied ri, rj, alpha, beta
- Excluded interactions (blacklist/translocations) are removed from both observed counts and the expectation calculation, not just deleted from the input
- Cscore output normalized so scores add up to 1 for each genomic region in the bedgraph

## Limitations

- The executable is compiled on a CentOS linux machine; if it does not run, rebuild from CscoreTool-M.cpp, twister.h and twister.cpp
- Blacklist.bed is used because encode blacklist regions are excluded, possibly due to copy number variations
- N_session must be chosen according to the resources available; the window bed must be an equal-length bed file with the Rabl position in the fourth column

## Evidence

- [readme] It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1).: "It should be an equal-length bed file with the fourth column being the Rabl position, i.e. the relative position between centromere (0) and telomere (1)."
- [methods] XXX_rabl.txt estimated Rabl effect, shown as a matrix.: "XXX_rabl.txt estimated Rabl effect, shown as a matrix."
- [results] In trans Hi-C interactions, CscoreTool-M models... the Rabl term Rij = 1 + alpha*ri*rj + beta*(ri+rj)*ri*rj and the observed reads Nij as a Poisson distribution with parameter Fij.: [evidence span withheld — anchor status: not_found]
- [intro] If the executable does not work, compile from source: CscoreTool-M.cpp, twister.h and twister.cpp with g++ using -fopenmp -O3: "g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M"
- [readme] Then any interactions between genomic regions in the same line will be excluded from the analysis. Note that the expectation are also excluded in the calculation.: "Then any interactions between genomic regions in the same line will be excluded from the analysis. Note that the expectation are also excluded in the calculation"
