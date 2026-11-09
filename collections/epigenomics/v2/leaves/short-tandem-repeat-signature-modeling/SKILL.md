---
name: short-tandem-repeat-signature-modeling
description: Use when benchmarking non-B DNA novelty detection (e.g. GoFAE-DND) on
  simulated data requires synthetic STR translocation-time windows that resemble experimental
  STR TT signals from nanopore reads.
when_to_use_negative:
- The input is real experimental nanopore translocation-time data (use the experimental
  window-preparation pipeline, not the simulator).
- You need synthetic G-quadruplex signatures rather than STR signatures (use the quadratic-profile
  G4 generator instead).
- Downstream models require real motif positions from the non-B DNA DB mapped to hg38
  rather than simulated benchmarks.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Python
  role: Implements simulator.py that generates the STR sinusoidal synthetic translocation-time
    windows
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Adds N(0,1) Gaussian noise to the sinusoidal STR translocation-time profile
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: pandas
  role: Writes the six synthetic labelled datasets to the simulated_data/ directory
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: matplotlib
  role: Inspects experimental G4 and STR signal shapes to tune the synthetic signature
    functions
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:18acad0961e97fb83908ffbd5947c5750176d658dfc9069d54c40336405a8623
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
yes_rate: 0.4
n_claims: 5
---

# short-tandem-repeat-signature-modeling

## Summary

Generates synthetic Short Tandem Repeat (STR) translocation-time windows by sampling a sinusoidal translocation-time profile perturbed with N(0,1) Gaussian noise, mimicking the experimental TT signatures observed for STR motifs in ONT nanopore sequencing of NA12878.

## When to use

Use when benchmarking non-B DNA novelty detection (e.g. GoFAE-DND) on simulated data requires synthetic STR translocation-time windows that resemble experimental STR TT signals from nanopore reads.

## When NOT to use

- The input is real experimental nanopore translocation-time data (use the experimental window-preparation pipeline, not the simulator).
- You need synthetic G-quadruplex signatures rather than STR signatures (use the quadratic-profile G4 generator instead).
- Downstream models require real motif positions from the non-B DNA DB mapped to hg38 rather than simulated benchmarks.

## Inputs

- Experimental translocation-time windows from task_001 (Tombo re-squiggled FAST5 events of NA12878, SRR15058166) used to tune the synthetic signature shape
- Baseline (non-signature) translocation-time distribution for B-DNA window generation

## Outputs

- Six labelled synthetic dataset files in simulated_data/ combining STR and G4 non-B windows with B-DNA windows at ratios 0.05, 0.1, 0.25
- Each dataset containing 200,000 B-DNA windows and 20,000 STR/G4 non-B windows with sinusoidal-plus-N(0,1)-noise TT profiles

## How to apply

First inspect experimental translocation-time windows produced from Tombo re-squiggled NA12878 reads (time duration computed as number of current measurements divided by the ONT sampling rate) to tune the shape of the STR synthetic signature. Implement the STR generator so non-B samples follow a sinusoidal translocation-time profile perturbed with N(0,1) Gaussian noise added via numpy, in contrast to the G-quadruplex generator which uses a quadratic profile. Assemble labelled datasets by combining these STR non-B windows with B-DNA windows drawn from the baseline (non-signature) translocation-time distribution at non-B-to-B ratios of 0.05, 0.1, and 0.25, each dataset containing 200,000 B-DNA and 20,000 non-B windows. Write the resulting datasets to the simulated_data/ directory using pandas via the simulator's output flag (e.g. -s ../simulated_data).

## Related tools

- **Python** (Implements simulator.py that generates the STR sinusoidal synthetic translocation-time windows) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Adds N(0,1) Gaussian noise to the sinusoidal STR translocation-time profile) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **pandas** (Writes the six synthetic labelled datasets to the simulated_data/ directory) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **matplotlib** (Inspects experimental G4 and STR signal shapes to tune the synthetic signature functions) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND

## Examples

```
cd simulator && python simulator.py -nb 10000 -b 1000000 -s ../simulated_data
```

## Evaluation signals

- Six dataset files exist in simulated_data/ after running the simulator with the -s flag.
- Each dataset contains exactly 200,000 B-DNA windows and 20,000 non-B windows (ratio 0.1 example shown in README commands).
- STR non-B windows follow a sinusoidal translocation-time profile plus N(0,1) Gaussian noise, visually distinguishable from the quadratic G4 profile via matplotlib inspection.
- Datasets cover the specified non-B-to-B labelled ratios of 0.05, 0.1, and 0.25 for both non-B types (STR and G4).

## Limitations

- The synthetic signature is a sinusoidal approximation tuned to resemble experimental TT signatures; it may not capture the full heterogeneity of real STR translocation-time behaviour.
- Gaussian N(0,1) noise is a simplifying assumption for the perturbation around the synthetic profile.
- Simulator runs may take a couple of minutes for large sample counts (e.g. 1,000,000 B-DNA windows).

## Evidence

- [other] The TT simulator generates STR samples from a sinusoidal function and G4 samples from a quadratic function, with N(0,1) Gaussian noise added, to resemble the experimental TT signatures.: [evidence span withheld — anchor status: not_found]
- [readme] README simulator invocation: "This is an example of command that simulate
100,000 B-DNA and 1,000 non-B DNA windows 
for G-quadruples and Short Tandem Repeat."
- [methods] Methods simulator command: "python simulator.py -nb 10000 -b 1000000 -s ../simulated_data"
- [methods] Translocation time computation: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
