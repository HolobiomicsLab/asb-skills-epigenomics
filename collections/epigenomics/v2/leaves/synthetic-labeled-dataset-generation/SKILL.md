---
name: synthetic-labeled-dataset-generation
description: Use when benchmarking non-B DNA detectors or classifiers on ONT nanopore
  translocation times and labelled synthetic windows with controlled non-B-to-B ratios
  are needed as ground truth.
when_to_use_negative:
- The goal is to evaluate on real sequencing signal rather than controlled synthetic
  benchmarks — use the experimental NA12878/SRR15058166 windows instead
- Input is already a labelled window dataset with the desired non-B-to-B ratio; regenerating
  with the simulator would duplicate it
- The downstream method requires other non-B structure types (e.g. Z-DNA, hairpins)
  not modeled by the quadratic/sinusoidal generators
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Python
  role: Language in which simulator.py is implemented and run
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Generates quadratic/sinusoidal TT profiles and N(0,1) Gaussian noise for non-B
    windows
- name: pandas
  role: Assembles and writes the six labelled synthetic datasets to simulated_data/
- name: matplotlib
  role: Used to inspect experimental G4 and STR TT signal shapes when tuning synthetic
    signatures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:b98bf5c265cd87e2d1e082fe05081128fed77b2e94271bc3ca56db413ea44afb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# synthetic-labeled-dataset-generation

## Summary

Reconstructs the translocation-time (TT) simulator that generates labelled synthetic benchmark datasets combining G-quadruplex and short-tandem-repeat non-B windows with B-DNA windows, so GoFAE-DND, classifiers, and novelty detectors can be evaluated on data with known labels.

## When to use

Use when benchmarking non-B DNA detectors or classifiers on ONT nanopore translocation times and labelled synthetic windows with controlled non-B-to-B ratios are needed as ground truth.

## When NOT to use

- The goal is to evaluate on real sequencing signal rather than controlled synthetic benchmarks — use the experimental NA12878/SRR15058166 windows instead
- Input is already a labelled window dataset with the desired non-B-to-B ratio; regenerating with the simulator would duplicate it
- The downstream method requires other non-B structure types (e.g. Z-DNA, hairpins) not modeled by the quadratic/sinusoidal generators

## Inputs

- Experimental translocation-time windows from task_001 (G-quadruplex and STR signals, window sizes 50 and 100) used to tune signature functions
- Signature-shape choices: quadratic function for G4, sinusoidal function for STR, both with N(0,1) Gaussian noise
- Baseline B-DNA translocation-time distribution (non-signature windows)

## Outputs

- Six labelled synthetic dataset files in simulated_data/, each containing 200,000 B-DNA and 20,000 non-B windows (G4 and STR types at non-B-to-B ratios of 0.05, 0.1, and 0.25)

## How to apply

First inspect experimental TT windows produced from Tombo re-squiggled NA12878 reads with pandas/matplotlib to tune synthetic signature shapes. Implement the simulator so G-quadruplex non-B samples follow a quadratic translocation-time profile plus N(0,1) Gaussian noise (numpy), while STR non-B samples follow a sinusoidal profile plus N(0,1) noise; draw B-DNA windows from the baseline non-signature TT distribution. Assemble six labelled datasets mixing the two non-B types at non-B-to-B ratios of 0.05, 0.1, and 0.25, each with 200,000 B-DNA and 20,000 non-B windows, and write them to simulated_data/ via the simulator's -s flag. Downstream tools consume these files with flags such as -nb 20000 -b 200000 and --nonb_ratio=0.1.

## Related tools

- **Python** (Language in which simulator.py is implemented and run) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Generates quadratic/sinusoidal TT profiles and N(0,1) Gaussian noise for non-B windows)
- **pandas** (Assembles and writes the six labelled synthetic datasets to simulated_data/)
- **matplotlib** (Used to inspect experimental G4 and STR TT signal shapes when tuning synthetic signatures)

## Examples

```
cd simulator && python simulator.py -nb 10000 -b 1000000 -s ../simulated_data
```

## Evaluation signals

- Six dataset files exist in simulated_data/ after running the simulator
- Each dataset contains exactly 200,000 B-DNA windows and 20,000 non-B windows
- The three non-B-to-B labelled ratios (0.05, 0.1, 0.25) are reproduced across G4 and STR dataset variants
- G4 non-B windows exhibit a quadratic TT profile and STR windows a sinusoidal profile, each perturbed with N(0,1) Gaussian noise, comparable to experimental TT signatures
- Downstream commands (e.g. classifier and novelty detector runs with -nb 20000 -b 200000) execute without errors against the generated files

## Limitations

- The simulator models only two non-B types (G-quadruplex via a quadratic function, STR via a sinusoidal function) with additive Gaussian noise, so synthetic signatures may not capture the full diversity of experimental non-B TT effects
- Simulation of a large run may take several minutes (README: 'This may take a couple minutes')

## Evidence

- [other] The TT simulator generates STR samples from a sinusoidal function and G4 samples from a quadratic function, with N(0,1) Gaussian noise added, to resemble the experimental TT signatures.: [evidence span withheld — anchor status: not_found]
- [other] assemble six labelled datasets combining G4 and STR non-B types at non-B-to-B ratios of 0.05, 0.1, and 0.25, each containing 200,000 B-DNA and 20,000 non-B windows: [evidence span withheld — anchor status: not_found]
- [methods] python simulator.py -nb 10000 -b 1000000 -s ../simulated_data: "python simulator.py -nb 10000 -b 1000000 -s ../simulated_data"
- [readme] This is an example of command that simulate 100,000 B-DNA and 1,000 non-B DNA windows for G-quadruples and Short Tandem Repeat. You must have matplotlib, numpy, and pandas installed.: "You must have matplotlib, numpy, and pandas installed."
- [other] Load the experimental translocation-time windows produced by task_001 and inspect the G-quadruplex and STR signal shapes with pandas/matplotlib to tune the synthetic signature functions.: [evidence span withheld — anchor status: not_found]
