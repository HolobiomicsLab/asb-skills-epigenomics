---
name: g-quadruplex-signature-modeling
description: Use when benchmarking non-B DNA detection from ONT translocation times
  and experimentally-derived G4/STR windows are unavailable or insufficient — synthetic
  TT datasets with known labels and controlled non-B-to-B ratios (0.05/0.1/0.25) are
  needed.
when_to_use_negative:
- Input is already experimental labelled translocation-time data (e.g. windows prepared
  from NA12878 reads around non-B DNA DB motifs) — the simulator is only needed for
  controlled synthetic benchmarks
- The downstream analysis targets real non-B discovery on new nanopore reads rather
  than evaluating novelty-detection performance against a known ground truth
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: Python
  role: Hosts simulator.py, which generates the synthetic G-quadruplex and STR translocation-time
    windows
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Generates the quadratic/sinusoidal TT signatures and adds N(0,1) Gaussian
    noise to non-B samples
- name: pandas
  role: Writes the six assembled synthetic datasets to the simulated_data/ output
    directory
- name: matplotlib
  role: Used with pandas to inspect experimental G-quadruplex and STR TT signal shapes
    to tune the synthetic signatures
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:5d705a1bb41ac5c3cf84b363516071944471b2c6037d04af1ac7cc223777a1a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# g-quadruplex-signature-modeling

## Summary

Generates synthetic G-quadruplex translocation-time (TT) windows from a quadratic signature perturbed with N(0,1) Gaussian noise, so that GoFAE-DND and competing novelty detectors/classifiers can be benchmarked against a controlled non-B signal. Part of the ONT-nonb-GoFAE-DND simulator, which assembles six labelled G4/STR datasets at non-B-to-B ratios of 0.05, 0.1, and 0.25.

## When to use

Use when benchmarking non-B DNA detection from ONT translocation times and experimentally-derived G4/STR windows are unavailable or insufficient — synthetic TT datasets with known labels and controlled non-B-to-B ratios (0.05/0.1/0.25) are needed.

## When NOT to use

- Input is already experimental labelled translocation-time data (e.g. windows prepared from NA12878 reads around non-B DNA DB motifs) — the simulator is only needed for controlled synthetic benchmarks
- The downstream analysis targets real non-B discovery on new nanopore reads rather than evaluating novelty-detection performance against a known ground truth

## Inputs

- Experimental translocation-time windows from NA12878 ONT sequencing (for tuning G4 and STR signature shapes)
- Target counts of non-B and B-DNA samples (e.g. -nb 20000, -b 200000)
- Non-B type specification: G-quadruplex (quadratic TT profile) and Short Tandem Repeat (sinusoidal TT profile)

## Outputs

- Six labelled synthetic datasets in simulated_data/, each with 200,000 B-DNA and 20,000 non-B windows (G4 and STR types at non-B-to-B ratios 0.05, 0.1, 0.25)
- Window-level TT matrices consumed by GoFAE-DND, novelty detectors, and classifiers via --sim_data_path

## How to apply

First inspect experimental translocation-time windows (produced by Tombo re-squiggle of NA12878 FAST5 files, with time duration computed as the ratio of current measurements to the ONT sampling rate) using pandas/matplotlib to tune the synthetic signature shapes. Implement the G-quadruplex generator in simulator.py so non-B windows follow a quadratic TT profile perturbed with numpy N(0,1) Gaussian noise (the Short Tandem Repeat generator instead uses a sinusoidal profile), while B-DNA windows are drawn from the baseline non-signature TT distribution. Run the simulator with explicit -nb (non-B) and -b (B-DNA) counts and an -s output directory, e.g. 200,000 B-DNA and 20,000 non-B windows per dataset, combining G4 and STR non-B types at ratios 0.05, 0.1, and 0.25, writing six files to simulated_data/ via pandas. Validate that six dataset files exist and each contains the expected 200,000 B-DNA and 20,000 non-B windows at the specified ratio before feeding them to Main.py with --data_type=simulated.

## Related tools

- **Python** (Hosts simulator.py, which generates the synthetic G-quadruplex and STR translocation-time windows) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Generates the quadratic/sinusoidal TT signatures and adds N(0,1) Gaussian noise to non-B samples)
- **pandas** (Writes the six assembled synthetic datasets to the simulated_data/ output directory)
- **matplotlib** (Used with pandas to inspect experimental G-quadruplex and STR TT signal shapes to tune the synthetic signatures)

## Examples

```
python simulator.py -nb 20000 -b 200000 -s ../simulated_data
```

## Evaluation signals

- Six dataset files exist in simulated_data/ after running the simulator
- Each dataset file contains exactly 200,000 B-DNA and 20,000 non-B windows
- Non-B-to-B ratios within each dataset match the specified 0.05, 0.1, or 0.25
- G4 non-B windows visually resemble a quadratic TT profile plus N(0,1) noise, in contrast to the sinusoidal STR profile
- Downstream GoFAE-DND runs (e.g. --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1) execute without file-format or label-ratio errors on the simulated data

## Limitations

- Synthetic signatures are simplified parametric shapes (quadratic for G4, sinusoidal for STR) plus Gaussian noise, tuned to resemble but not reproduce experimental TT signatures
- The simulator enforces fixed sample counts (e.g. 200,000 B-DNA per dataset), so extreme class imbalance beyond the predefined 0.05–0.25 non-B ratios requires regenerating datasets

## Evidence

- [other] The TT simulator generates STR samples from a sinusoidal function and G4 samples from a quadratic function, with N(0,1) Gaussian noise added, to resemble the experimental TT signatures.: [evidence span withheld — anchor status: not_found]
- [readme] $ python simulator.py -nb 10000 -b 1000000 -s ../simulated_data: "python simulator.py -nb 10000 -b 1000000 -s ../simulated_data"
- [readme] This is an example of command that simulate 100,000 B-DNA and 1,000 non-B DNA windows for G-quadruples and Short Tandem Repeat. You must have matplotlib, numpy, and pandas installed.: "You must have matplotlib, numpy, and pandas installed."
- [methods] we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
