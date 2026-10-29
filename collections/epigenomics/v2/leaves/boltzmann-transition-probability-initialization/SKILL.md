---
name: boltzmann-transition-probability-initialization
description: Use when training RoboCOP's multivariate HMM to compute genome-wide probabilistic
  occupancy of nucleosomes and ~150 TFs from MNase-seq or ATAC-seq accessibility plus
  nucleotide sequence, and you must initialize the transition matrix before Baum-Welch
  learning.
when_to_use_negative:
- Do not use as a generic HMM initializer for unrelated models — the two-standard-deviation
  cap on TF transitions is specific to RoboCOP's multivariate chromatin occupancy
  HMM and its ~150-TF state space.
- Do not reach for this when the analysis goal is peak calling plus TF labeling (MACS2
  + FIMO alternative) rather than joint probabilistic occupancy computation via a
  hidden Markov model.
- Do not expect prior/experimentally measured transition probabilities to be incorporated
  — the article explicitly defers incorporation of prior DBF transition information
  to future work.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: RoboCOP
  role: Multivariate HMM whose transition matrix is Boltzmann-initialized and learned
    via Baum-Welch; compiled into librobocop.so and configured via config.ini
  repo: https://github.com/HarteminkLab/RoboCOP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab553
    title: 'RoboCOP: jointly computing chromatin occupancy profiles for numerous factors
      from chromatin accessibility data'
  source_content_hash: sha256:0c6c6dd29355d9061d6246f5b57f6275cc9864bd478781c504f4255491add2b7
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/HarteminkLab/RoboCOP
  tools:
  - RoboCOP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# boltzmann-transition-probability-initialization

## Summary

Initializes the transition probabilities of RoboCOP's multivariate hidden Markov model with a Boltzmann distribution, providing the starting point for Baum-Welch EM learning of TF and nucleosome transition probabilities in the joint chromatin occupancy model.

## When to use

Use when training RoboCOP's multivariate HMM to compute genome-wide probabilistic occupancy of nucleosomes and ~150 TFs from MNase-seq or ATAC-seq accessibility plus nucleotide sequence, and you must initialize the transition matrix before Baum-Welch learning.

## When NOT to use

- Do not use as a generic HMM initializer for unrelated models — the two-standard-deviation cap on TF transitions is specific to RoboCOP's multivariate chromatin occupancy HMM and its ~150-TF state space.
- Do not reach for this when the analysis goal is peak calling plus TF labeling (MACS2 + FIMO alternative) rather than joint probabilistic occupancy computation via a hidden Markov model.
- Do not expect prior/experimentally measured transition probabilities to be incorporated — the article explicitly defers incorporation of prior DBF transition information to future work.

## Inputs

- Initial transition probability vector over RoboCOP HMM states (nucleosomes and TF DBFs) for Boltzmann initialization
- MNase-seq or ATAC-seq paired-end fragment midpoint counts per genomic position
- Nucleotide sequence for PWM/dinucleotide sequence emission models
- config.ini with cshared path pointing to the compiled librobocop.so

## Outputs

- Boltzmann-initialized transition probability matrix serving as the starting point for Baum-Welch EM
- Learned transition probabilities for nucleosome and TF states, with TF transitions capped at two standard deviations above the mean of initial values
- Genome-wide probabilistic chromatin occupancy profile at single-nucleotide resolution

## How to apply

Within RoboCOP's HMM training (model core compiled into librobocop.so from the HarteminkLab/RoboCOP repo), initialize the transition probabilities of all DNA-binding factors (DBFs) — nucleosomes plus ~150 transcription factors — using a Boltzmann distribution over states. These initial values then seed Baum-Welch EM updates of the transition matrix. A key modification applies during learning: TF transition probabilities are capped at a threshold δ chosen to be two standard deviations above the mean of the initial (Boltzmann-distributed) transition probabilities of all TFs, preventing EM updates from inflating individual TF transition probabilities. The sequence emission side uses PWM and dinucleotide emission models while accessibility emissions use negative-binomial distributions over fragment midpoint counts, all estimated jointly with the transitions. Rationale: Boltzmann initialization spreads probability mass across DBF states before any data-driven refinement, and the 2-sd cap enforces a ceiling on learned TF persistence in the absence of prior transition information.

## Related tools

- **RoboCOP** (Multivariate HMM whose transition matrix is Boltzmann-initialized and learned via Baum-Welch; compiled into librobocop.so and configured via config.ini) — https://github.com/HarteminkLab/RoboCOP

## Examples

```
cd RoboCOP/robocop && bash gccCompile && ls "`pwd`/librobocop.so"
```

## Evaluation signals

- Boltzmann initialization produces a normalized transition probability distribution over the HMM's nucleosome and TF states before any EM iteration.
- After Baum-Welch convergence, no individual TF transition probability exceeds the threshold δ = mean(initial TF transitions) + 2 × std(initial TF transitions).
- librobocop.so exists at the absolute path printed by `ls \`pwd\`/librobocop.so` and the cshared entry in config.ini resolves to that file.
- Resulting occupancy profile covers the genome at single-nucleotide resolution for nucleosomes plus ~150 TFs, and TF binding site predictions outperform the COMPETE baseline.
- Run example_robocop.ipynb vignette end-to-end with config_example.ini as a template to confirm the initialized model trains without errors.

## Limitations

- The 2-sd cap on TF transition probabilities is a heuristic; no prior information about DBF transition probabilities is incorporated (deferred to future work by the authors).
- MNase-seq yields markedly better RoboCOP performance than ATAC-seq, so initialized models trained on ATAC-seq data may be less accurate.
- Accuracy of the learned transitions and downstream occupancy profiles improves with deeper sequencing; shallow libraries degrade estimates.
- The model core must be compiled (bash gccCompile) and librobocop.so registered in config.ini before initialization and training can run.

## Evidence

- [intro] uses a multivariate hidden Markov model (HMM) (23) to compute a probabilistic occupancy landscape of hundreds of DBFs genome-wide at single-nucleotide resolution: "uses a multivariate hidden Markov model (HMM) (23) to compute a probabilistic occupancy landscape of hundreds of DBFs genome-wide at single-nucleotide resolution"
- [other] Initialize transition probabilities with a Boltzmann distribution, then implement Baum-Welch EM updates with the modification that caps TF transition probabilities: [evidence span withheld — anchor status: not_found]
- [other] the threshold δ is chosen to be two standard deviations above the mean of the initial transition probabilities of all the TFs: [evidence span withheld — anchor status: not_found]
- [readme] bash gccCompile # generates shared library librobocop.so: "bash gccCompile # generates shared library librobocop.so"
- [discussion] In future work, it might be possible to improve TF binding site predictions through the incorporation of prior information about DBF transition probabilities: "In future work, it might be possible to improve TF binding site predictions through the incorporation of prior information about DBF transition probabilities"
