---
name: non-b-dna-simulation-benchmarking
description: Use when evaluating or comparing non-B DNA detection methods on simulated
  ONT translocation-time windows across non-B ratios (e.g. 0.05), needing F1 benchmark
  tables like Fig. 6 / Supp. Table S6.
when_to_use_negative:
- Working with experimental NA12878 ONT data instead of the simulator output — use
  the FDR-controlled novelty detection comparison (Supp. Table S5) workflow instead
- The goal is to train a supervised classifier for production prediction rather than
  benchmark novelty detection across class-imbalance ratios
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: GoFAE-DND
  role: Novelty detection autoencoder with goodness-of-fit regularizer; run with --data_type=simulated,
    n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35,
    lambda_alpha=0.5
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: isolation_forest.py
  role: Baseline novelty detector run on simulated data with -nb 20000 -b 200000
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: local_outlier_factor.py
  role: Baseline novelty detector run on simulated data with -nb 20000 -b 200000
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: svm_one_class.py
  role: One-class SVM baseline novelty detector run on simulated data with -nb 20000
    -b 200000
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: simulator.py
  role: Generates simulated B-DNA and non-B DNA translocation-time windows (e.g. -nb
    10000 -b 1000000)
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: Python
  role: Runtime for all pipeline scripts; requires matplotlib, numpy, pandas, seaborn,
    h5py
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:f1059931cd2b666859811af3ea12cd1a51ae4cfc657e88a0227effa3c9b62407
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - GoFAE-DND
  - Python
  - conda
  - h5py
  - isolation_forest.py
  - local_outlier_factor.py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  - simulator.py
  - svm_one_class.py
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.631578947368421
n_claims: 19
---

# non-b-dna-simulation-benchmarking

## Summary

Benchmark GoFAE-DND against novelty detectors (IF, LOF, one-class SVM) and supervised classifiers (SVC, logistic regression, nearest neighbors, Gaussian process, random forest) on simulated nanopore translocation-time windows, computing F1 scores across non-B to B-DNA ratios to reproduce Fig. 6 / Supp. Table S6.

## When to use

Use when evaluating or comparing non-B DNA detection methods on simulated ONT translocation-time windows across non-B ratios (e.g. 0.05), needing F1 benchmark tables like Fig. 6 / Supp. Table S6.

## When NOT to use

- Working with experimental NA12878 ONT data instead of the simulator output — use the FDR-controlled novelty detection comparison (Supp. Table S5) workflow instead
- The goal is to train a supervised classifier for production prediction rather than benchmark novelty detection across class-imbalance ratios

## Inputs

- Simulated translocation-time windows for G-quadruplex and short-tandem-repeat motifs with matched B-DNA controls (from simulator.py)
- Non-B to B-DNA ratio settings (e.g. 0.05 most challenging ratio)
- Simulated data directory (simulated_data/) in h5py-readable format

## Outputs

- F1 benchmark table for all method x non-B type x ratio combinations (Supp. Table S6 equivalent)
- Metric spreadsheets and diagnostic images in the root-level results/ directory
- Fig. 6-style comparison of GoFAE-DND vs SVM, LOF, IF, and supervised classifiers

## How to apply

Generate simulated translocation-time windows with simulator.py (e.g. -nb 10000 -b 1000000), then run GoFAE-DND via Main.py --data_type=simulated with n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5 for each non-B type (G_Quadruplex_Motif, Short_tandem_repeat) and non-B ratio. Run the novelty detectors isolation_forest.py, local_outlier_factor.py, and svm_one_class.py on the simulated data with -nb 20000 -b 200000, then train classifiers svc.py, random_forest.py, nearest_neighbors.py, logistic_regression.py, and gaussian_process.py on the same splits. Compute F1 scores for every method x dataset x ratio combination and compile a benchmark table, verifying GoFAE-DND ranks above SVM, LOF, and IF, and that novelty detectors outperform the supervised classifiers.

## Related tools

- **GoFAE-DND** (Novelty detection autoencoder with goodness-of-fit regularizer; run with --data_type=simulated, n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **isolation_forest.py** (Baseline novelty detector run on simulated data with -nb 20000 -b 200000) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **local_outlier_factor.py** (Baseline novelty detector run on simulated data with -nb 20000 -b 200000) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **svm_one_class.py** (One-class SVM baseline novelty detector run on simulated data with -nb 20000 -b 200000) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **simulator.py** (Generates simulated B-DNA and non-B DNA translocation-time windows (e.g. -nb 10000 -b 1000000)) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **Python** (Runtime for all pipeline scripts; requires matplotlib, numpy, pandas, seaborn, h5py)

## Examples

```
cd simulator && python simulator.py -nb 10000 -b 1000000 -s ../simulated_data; cd ../GoFAE-DND/gofaednd && python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- At non-B ratio 0.05, GoFAE-DND F1 should reproduce the reported values: 0.2812 for G-quadruplex and 0.2344 for short tandem repeat
- Method ordering at the hardest ratio should be GoFAE-DND > SVM > LOF > IF
- Novelty detectors (GoFAE-DND, SVM, LOF, IF) should outperform the supervised classifiers on the imbalanced simulated datasets
- Results spreadsheets and diagnostic images should appear in the root-level results/ directory

## Limitations

- F1 scores at the most challenging 0.05 non-B ratio are low overall (best ~0.28), reflecting extreme class imbalance inherent to the benchmark
- Only G-quadruplex and short tandem repeat motifs are covered by the simulation example; other non-B types require additional simulator configurations
- The simulator run and GoFAE-DND training can be slow ('This may take a couple minutes' for simulation; multi-epoch training per ratio)

## Evidence

- [results] GoFAE-DND achieved the highest F1 scores (0.2812 and 0.2344) for the most challenging non-B ratio (0.05): [evidence span withheld — anchor status: not_found]
- [readme] python simulator.py -nb 10000 -b 1000000 -s ../simulated_data: "$ python simulator.py -nb 10000 -b 1000000 -s ../simulated_data"
- [readme] python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5: "$ python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2"
- [readme] python isolation_forest.py -W ignore -d sim -f ../simulated_data/ -r ../results/ -nb 20000 -b 200000: "$ python isolation_forest.py -W ignore -d sim -f ../simulated_data/ -r ../results/ -nb 20000 -b 200000"
- [readme] The results for both novelty detectors and classifiers can be found in the root-level results directory.: "The results for both novelty detectors and classifiers can be found in the root-level
results directory."
