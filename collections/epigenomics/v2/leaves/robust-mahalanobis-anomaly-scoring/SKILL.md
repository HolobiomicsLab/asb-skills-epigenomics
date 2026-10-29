---
name: robust-mahalanobis-anomaly-scoring
description: Use when detecting non-B DNA windows in nanopore translocation-time matrices
  as a novelty detection problem, or when scoring windows against an empirical null
  built from B-DNA controls.
when_to_use_negative:
- Input is raw ONT current or FAST5 signal that has not yet been converted to base-level
  translocation times via Albacore basecalling and Tombo re-squiggling
- The goal is supervised classification with labeled non-B training data (the repo
  instead offers SVC, random forest, nearest neighbors, logistic regression, and gaussian
  process classifiers)
- Windows have not been prepared around non-B DB motifs with fixed-length windows,
  opposite-strand extension, and motif-free control regions
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Runs Main.py to train GoFAE-DND, compute novelty statistics, calibrate P-values,
    and apply Benjamini-Hochberg FDR
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Numerical computation of robust Mahalanobis-distance and reconstruction-error
    novelty statistics
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: pandas
  role: Writes the novelty-score table and metric spreadsheets to the results/ directory
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: matplotlib
  role: Generates diagnostic images accompanying the metric spreadsheets
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:b68d96f0860dfc578da19d06eb13c222f5f846b4896b4e3f2173e3d59063e890
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# Robust Mahalanobis anomaly scoring

## Summary

Computes robust Mahalanobis-distance (and reconstruction-error) novelty statistics from the GoFAE-DND autoencoder's latent encoding of ONT translocation-time windows, calibrates empirical-null P-values from held-out B-DNA windows, and calls non-B DNA via Benjamini-Hochberg FDR.

## When to use

Use when detecting non-B DNA windows in nanopore translocation-time matrices as a novelty detection problem, or when scoring windows against an empirical null built from B-DNA controls.

## When NOT to use

- Input is raw ONT current or FAST5 signal that has not yet been converted to base-level translocation times via Albacore basecalling and Tombo re-squiggling
- The goal is supervised classification with labeled non-B training data (the repo instead offers SVC, random forest, nearest neighbors, logistic regression, and gaussian process classifiers)
- Windows have not been prepared around non-B DB motifs with fixed-length windows, opposite-strand extension, and motif-free control regions

## Inputs

- Translocation-time matrix of windows from upstream window-extraction step (experimental NA12878 or simulated data, window sizes 50 and 100)
- Windows split into B-DNA training, held-out B-DNA (empirical null), and non-B DNA test sets at nonb_ratio=0.1
- GoFAE-DND configuration (n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5)

## Outputs

- Novelty-score table with scores, empirical-null P-values, and FDR-0.2 non-B calls
- Trained GoFAE-DND model
- Spreadsheets of metrics and diagnostic images in the root-level results/ directory

## How to apply

Train the GoFAE-DND autoencoder on B-DNA windows with latent dimension n_z=64, 64 random projections, 25 epochs, discriminative_weight=35, lambda_alpha=0.5, and a Gaussian goodness-of-fit (Shapiro-Wilk) test on the encoding as regularizer, so the discriminative loss drives poor reconstruction of non-B-like inputs. Split windows at nonb_ratio=0.1 into B-DNA training, held-out B-DNA (empirical null), and non-B test sets. For each held-out window compute reconstruction-error and robust Mahalanobis-distance novelty statistics from the trained encoder, then calibrate empirical null P-values using the held-out B-DNA windows. Apply Benjamini-Hochberg multiple-testing correction at fdr_level=0.2 to issue final non-B novelty calls, and write the novelty-score table (scores, P-values, calls) plus model and diagnostic spreadsheets to the root-level results/ directory.

## Related tools

- **Python** (Runs Main.py to train GoFAE-DND, compute novelty statistics, calibrate P-values, and apply Benjamini-Hochberg FDR) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Numerical computation of robust Mahalanobis-distance and reconstruction-error novelty statistics) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **pandas** (Writes the novelty-score table and metric spreadsheets to the results/ directory) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **matplotlib** (Generates diagnostic images accompanying the metric spreadsheets) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- The root-level results/ directory contains spreadsheets of metrics and diagnostic images after a run
- Reported P-value calls respect the configured FDR level of 0.2 under Benjamini-Hochberg correction
- Window splits respect the configured non-B ratio of 0.1 (B-DNA training vs held-out null vs non-B test)
- Model hyperparameters match the documented run: n_z=64, num_projections=64, epochs=25, discriminative_weight=35, lambda_alpha=0.5
- Comparison baselines (isolation forest, local outlier factor, one-class SVM) run on the same simulated_data/ input for relative performance checks

## Limitations

- The empirical null is calibrated only from held-out B-DNA windows, so contaminating non-B structure in the 'B-DNA' set would skew P-values
- Computational methods rely on non-B DNA base motifs, which are necessary but not sufficient indicators of non-B structures
- Translocation times are computed as the ratio of the number of current measurements to the ONT sampling rate, inheriting assumptions from Tombo re-squiggling and the reference genome (hg38)
- Downloaded FAST5 experimental data total over 4TB, making full reproduction slow

## Evidence

- [abstract] We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer.: "We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer."
- [abstract] A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures.: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures."
- [readme] $ python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1: "$ python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1"
- [readme] The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory.: "The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory."
- [readme] For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate.: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
