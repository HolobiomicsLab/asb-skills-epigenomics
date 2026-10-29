---
name: classifier-versus-novelty-detector-evaluation
description: Use when reproducing or extending the simulated translocation-time benchmark
  (Fig. 6, Supp. Table S6) that compares novelty detectors and supervised classifiers
  for non-B DNA detection at varying non-B ratios.
when_to_use_negative:
- Working on the experimental NA12878 nanopore data pipeline (Albacore/Tombo processing)
  rather than the simulated benchmark — the detector/classifier comparison scripts
  here are invoked with -d sim on simulated_data/
- Input is already a compiled F1 table and the goal is only re-plotting; use the comparison/plotting
  step instead of re-running all detectors and classifiers
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: GoFAE-DND
  role: Novelty-detection autoencoder (Main.py --data_type=simulated) whose F1 scores
    are benchmarked against competing methods
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: isolation_forest.py
  role: Baseline novelty detector run on simulated data with -nb 20000 -b 200000 to
    produce anomaly scores for the comparison
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: local_outlier_factor.py
  role: Baseline novelty detector (LOF) run on simulated translocation-time windows
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: svm_one_class.py
  role: One-class SVM baseline novelty detector run on simulated data
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: svc.py / random_forest.py / nearest_neighbors.py / logistic_regression.py
    / gaussian_process.py
  role: Supervised classifiers trained on the same simulated splits to show novelty
    detectors outperform classification models
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: simulator.py
  role: Generates the simulated non-B and B-DNA translocation-time windows (e.g. -nb
    10000 -b 1000000) that feed the whole benchmark
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy / pandas / matplotlib / seaborn / h5py
  role: Dependencies for running the simulator, novelty detectors, classifiers, and
    compiling/plots of the F1 benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:125e13dfdb1695fdc870983de578d1bf5173d1a419a4c3af3860fd48f12b02ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - GoFAE-DND
  - Python
  - h5py
  - isolation_forest.py
  - local_outlier_factor.py
  - matplotlib
  - numpy
  - numpy / pandas / matplotlib / seaborn / h5py
  - pandas
  - seaborn
  - simulator.py
  - svc.py / random_forest.py / nearest_neighbors.py / logistic_regression.py / gaussian_process.py
  - svm_one_class.py
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# classifier-versus-novelty-detector-evaluation

## Summary

Benchmark F1 scores of the GoFAE-DND novelty detector against other novelty detection methods (one-class SVM, LOF, isolation forest) and supervised classifiers (SVC, random forest, nearest neighbors, logistic regression, gaussian process) across non-B to B-DNA ratios on simulated ONT translocation-time windows.

## When to use

Use when reproducing or extending the simulated translocation-time benchmark (Fig. 6, Supp. Table S6) that compares novelty detectors and supervised classifiers for non-B DNA detection at varying non-B ratios.

## When NOT to use

- Working on the experimental NA12878 nanopore data pipeline (Albacore/Tombo processing) rather than the simulated benchmark — the detector/classifier comparison scripts here are invoked with -d sim on simulated_data/
- Input is already a compiled F1 table and the goal is only re-plotting; use the comparison/plotting step instead of re-running all detectors and classifiers

## Inputs

- Simulated translocation-time windows for G-quadruplex and Short_tandem_repeat motifs with matched B-DNA controls (from simulator.py, stored in simulated_data/)
- Non-B to B-DNA ratio settings (e.g. down to the most challenging 0.05 ratio)
- Trained-method outputs: novelty scores from GoFAE-DND, IF, LOF, one-class SVM; predictions from SVC, random forest, nearest neighbors, logistic regression, gaussian process

## Outputs

- Benchmark table of F1 scores for every method x dataset (G-quadruplex, STR) x non-B ratio combination
- Metric spreadsheets and diagnostic images in the root-level results/ directory
- Figure 6 / Supp. Table S6 style comparison plots (e.g. via plottings.py)

## How to apply

First generate simulated G-quadruplex and short-tandem-repeat translocation-time windows with matched B-DNA controls via the repository's simulator (e.g. -nb 10000 -b 1000000). Run GoFAE-DND (python Main.py --data_type=simulated) with n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5 for each non-B type and ratio to obtain novelty scores and metric spreadsheets in results/. Then run the novelty detectors isolation_forest.py, local_outlier_factor.py, and svm_one_class.py with -nb 20000 -b 200000, and train the classifiers svc.py, random_forest.py, nearest_neighbors.py, logistic_regression.py, and gaussian_process.py on the same simulated splits. Compute F1 scores for every method x dataset x ratio combination into a single benchmark table and check the expected ordering: GoFAE-DND highest, then SVM, LOF, IF, with novelty detectors outperforming supervised classifiers.

## Related tools

- **GoFAE-DND** (Novelty-detection autoencoder (Main.py --data_type=simulated) whose F1 scores are benchmarked against competing methods) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **isolation_forest.py** (Baseline novelty detector run on simulated data with -nb 20000 -b 200000 to produce anomaly scores for the comparison) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **local_outlier_factor.py** (Baseline novelty detector (LOF) run on simulated translocation-time windows) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **svm_one_class.py** (One-class SVM baseline novelty detector run on simulated data) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **svc.py / random_forest.py / nearest_neighbors.py / logistic_regression.py / gaussian_process.py** (Supervised classifiers trained on the same simulated splits to show novelty detectors outperform classification models) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **simulator.py** (Generates the simulated non-B and B-DNA translocation-time windows (e.g. -nb 10000 -b 1000000) that feed the whole benchmark) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy / pandas / matplotlib / seaborn / h5py** (Dependencies for running the simulator, novelty detectors, classifiers, and compiling/plots of the F1 benchmark)

## Examples

```
cd GoFAE-DND/gofaednd && python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.05 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- At the most challenging non-B ratio 0.05, the reproduced table shows GoFAE-DND F1 of 0.2812 for G-quadruplex and 0.2344 for short tandem repeat
- Method ordering matches the paper: GoFAE-DND highest by a wide margin, followed by SVM, then LOF, then IF
- Novelty detectors outperform the supervised classifiers across the simulated datasets
- All per-method metric spreadsheets and diagnostic images are written into the root-level results/ directory
- GoFAE-DND runs use the reported hyperparameters (n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5)

## Limitations

- F1 scores at the hardest ratio (0.05) are low in absolute terms (0.23-0.28), so performance should be judged relatively across methods rather than by absolute value
- The benchmark depends on the simulator's synthetic translocation times; conclusions may not transfer directly to experimental NA12878 data
- The comparison/plotting step (plottings.py to reproduce Fig 6 and S4) appears commented out in the README and may need manual re-enabling

## Evidence

- [results] Run the novelty detectors isolation_forest.py, local_outlier_factor.py and svm_one_class.py on the simulated data (nb 20000, b 200000) to obtain their anomaly scores.: [evidence span withheld — anchor status: not_found]
- [readme] python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5: "python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2"
- [readme] we can run isolation forests, local outlier factors, and one class SVMs.: "Afterwards, we can run isolation forests, local outlier factors, and one class SVMs."
- [readme] python -W ignore svc.py -d sim -f ../simulated_data/ -r ../results/ -nb 20000 -b 200000: "$ python -W ignore svc.py -d sim -f ../simulated_data/ -r ../results/ -nb 20000 -b 200000"
- [methods] python simulator.py -nb 10000 -b 1000000 -s ../simulated_data: "python simulator.py -nb 10000 -b 1000000 -s ../simulated_data"
