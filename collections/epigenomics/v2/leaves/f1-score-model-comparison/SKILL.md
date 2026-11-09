---
name: f1-score-model-comparison
description: Use when comparing a novelty detection model (e.g. GoFAE-DND) against
  other anomaly detectors and supervised classifiers using F1 scores across imbalanced
  class ratios of simulated data.
when_to_use_negative:
- Input is experimental nanopore data rather than simulator output — the benchmark
  F1 values (0.2812, 0.2344) apply to the simulated G4/STR datasets
- The goal is non-B structure prediction itself, not method benchmarking — use the
  GoFAE-DND prediction workflow instead
- A balanced dataset is available, since the benchmark is specifically designed for
  heavily imbalanced non-B:B ratios down to 0.05
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0091
tools:
- name: GoFAE-DND
  role: Autoencoder novelty detector with goodness-of-fit regularization whose F1
    scores are compared against alternatives
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: Python
  role: Language for running the simulator, detectors, classifiers, and benchmark
    compilation
- name: numpy
  role: Numerical dependency for simulator and analysis scripts
- name: pandas
  role: Metric spreadsheets and benchmark table compilation
- name: matplotlib
  role: Plotting the method comparison figures (Fig. 6)
- name: seaborn
  role: Dependency for novelty detector scripts
- name: h5py
  role: Dependency for novelty detector scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:d952f4ef9e64673bf18c2e65d556f18f5182d70494a62870915dac4bc5622e49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - GoFAE-DND
  - Python
  - h5py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# f1-score-model-comparison

## Summary

Benchmark F1 scores of GoFAE-DND novelty detection against SVM, LOF, isolation forest, and supervised classifiers across non-B to B-DNA ratios on simulated ONT translocation-time windows, reproducing the comparison behind Fig. 6 and Supp. Table S6.

## When to use

Use when comparing a novelty detection model (e.g. GoFAE-DND) against other anomaly detectors and supervised classifiers using F1 scores across imbalanced class ratios of simulated data.

## When NOT to use

- Input is experimental nanopore data rather than simulator output — the benchmark F1 values (0.2812, 0.2344) apply to the simulated G4/STR datasets
- The goal is non-B structure prediction itself, not method benchmarking — use the GoFAE-DND prediction workflow instead
- A balanced dataset is available, since the benchmark is specifically designed for heavily imbalanced non-B:B ratios down to 0.05

## Inputs

- Simulated translocation-time windows for G-quadruplex and short tandem repeat motifs with matched B-DNA controls (simulated_data/)
- Non-B to B-DNA ratios including the most challenging 0.05 ratio
- GoFAE-DND metric spreadsheets from results/ directory

## Outputs

- F1 benchmark table over methods x datasets x non-B ratios
- Method ranking (GoFAE-DND > SVM > LOF > IF; novelty detectors > supervised classifiers)
- Comparison figures reproducing Fig. 6 (plottings.py)

## How to apply

Simulate translocation-time windows with the repository's simulator (e.g. python simulator.py -nb 10000 -b 1000000), then run GoFAE-DND per non-B type and ratio with n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5. Run the novelty detectors isolation_forest.py, local_outlier_factor.py, and svm_one_class.py, and the classifiers svc.py, random_forest.py, nearest_neighbors.py, logistic_regression.py, and gaussian_process.py on the same simulated splits (nb 20000, b 200000). Compute F1 for every method x dataset x ratio combination and verify GoFAE-DND ranks highest, followed by SVM, LOF, then IF, with novelty detectors outperforming supervised classifiers. At the hardest ratio 0.05, check the F1 values match the reported 0.2812 (G-quadruplex) and 0.2344 (short tandem repeat).

## Related tools

- **GoFAE-DND** (Autoencoder novelty detector with goodness-of-fit regularization whose F1 scores are compared against alternatives) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **Python** (Language for running the simulator, detectors, classifiers, and benchmark compilation)
- **numpy** (Numerical dependency for simulator and analysis scripts)
- **pandas** (Metric spreadsheets and benchmark table compilation)
- **matplotlib** (Plotting the method comparison figures (Fig. 6))
- **seaborn** (Dependency for novelty detector scripts)
- **h5py** (Dependency for novelty detector scripts)

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.05 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- At non-B ratio 0.05, GoFAE-DND F1 equals 0.2812 for G-quadruplex and 0.2344 for short tandem repeat
- F1 ordering is GoFAE-DND > SVM > LOF > IF across ratios
- Novelty detectors outperform the supervised classification models (SVC, logistic regression, nearest neighbors, gaussian process, random forest)
- Metric spreadsheets and diagnostic images are written to the root-level results/ directory for every method x ratio run

## Limitations

- F1 scores at the hardest ratio are low (~0.23-0.28), reflecting the extreme class imbalance of 0.05 non-B:B
- Benchmark conclusions are based on simulated translocation times from the paper's simulator, not directly on experimental ONT data
- Novelty detectors require seaborn and h5py installed, and classifiers/detectors use fixed sample sizes (-nb 20000 -b 200000)

## Evidence

- [other] GoFAE-DND achieved the highest F1 scores (0.2812 and 0.2344) for the most challenging non-B ratio (0.05): [evidence span withheld — anchor status: not_found]
- [readme] run GoFAE-DND with specific hyperparameters: "python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25"
- [readme] novelty detectors on simulated data: "python isolation_forest.py -W ignore -d sim -f ../simulated_data/ -r ../results/ -nb 20000 -b 200000"
- [methods] classifiers on simulated data: "python -W ignore svc.py -d sim -f ../simulated_data/ -r ../results/ -nb 20000 -b 200000"
- [readme] simulator command: "This is an example of command that simulate 100,000 B-DNA and 1,000 non-B DNA windows for G-quadruples and Short Tandem Repeat."
