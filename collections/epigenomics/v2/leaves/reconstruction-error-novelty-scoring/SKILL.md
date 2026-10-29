---
name: reconstruction-error-novelty-scoring
description: Use when translocation-time windows (B-DNA and candidate non-B) from
  ONT reads are available and the goal is novelty-based non-B DNA detection with per-window
  P-values and FDR-controlled calls.
when_to_use_negative:
- The question is supervised classification of known non-B motifs — use the classifiers
  (SVC, random forest, nearest neighbors, logistic regression, gaussian process) instead
  of novelty scoring
- Input is raw nanopore current/FAST5 signal that has not yet been basecalled (Albacore),
  re-squiggled (Tombo), and converted into per-base translocation times
- Windows have not been defined around motifs from the non-B DNA DB and motif-free
  controls identified
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: Python
  role: Implements Main.py, which trains the GoFAE-DND autoencoder, computes novelty
    statistics and empirical-null P-values, and applies Benjamini-Hochberg FDR
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Numerical computation of reconstruction errors, robust Mahalanobis distances,
    and random projections
- name: pandas
  role: Writes the novelty-score and metric spreadsheets to the results/ directory
- name: matplotlib
  role: Generates the diagnostic images accompanying the results spreadsheets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:d031829c5b65814fdafa0a53232560fd95cd2f49a3e6aacb1808e3257455e816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - matplotlib
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# reconstruction-error-novelty-scoring

## Summary

Scores non-B DNA windows from nanopore translocation-time matrices by reconstruction error and robust Mahalanobis distance from a GoFAE-DND autoencoder, calibrating empirical-null P-values against held-out B-DNA windows and calling non-B novelty via Benjamini-Hochberg FDR at 0.2.

## When to use

Use when translocation-time windows (B-DNA and candidate non-B) from ONT reads are available and the goal is novelty-based non-B DNA detection with per-window P-values and FDR-controlled calls.

## When NOT to use

- The question is supervised classification of known non-B motifs — use the classifiers (SVC, random forest, nearest neighbors, logistic regression, gaussian process) instead of novelty scoring
- Input is raw nanopore current/FAST5 signal that has not yet been basecalled (Albacore), re-squiggled (Tombo), and converted into per-base translocation times
- Windows have not been defined around motifs from the non-B DNA DB and motif-free controls identified

## Inputs

- Translocation-time matrix of B-DNA and non-B DNA windows produced by upstream window extraction (window sizes 50 and 100; experimental or simulated data)
- Held-out B-DNA test windows used to build the empirical null distribution
- Trained GoFAE-DND encoder (n_z=64) with 64 random projections

## Outputs

- Novelty-score table with per-window scores, empirical-null P-values, and FDR-level calls
- Trained GoFAE-DND model artifact
- Spreadsheets of metrics and diagnostic images in the root-level results/ directory

## How to apply

Load the translocation-time matrix from the window-extraction step and split windows into B-DNA training, held-out B-DNA (empirical null), and non-B test sets at the configured non-B ratio of 0.1. Train the GoFAE-DND autoencoder on B-DNA windows with latent dimension n_z=64, 64 random projections, 25 epochs, discriminative loss weight 35, and lambda_alpha=0.5, using a Gaussian goodness-of-fit (Shapiro-Wilk) test on the encoding as a regularizer so the discriminative loss drives poor reconstruction of non-B-like inputs. For each held-out window, compute reconstruction-error and robust Mahalanobis-distance novelty statistics from the trained encoder. Calibrate empirical-null P-values for these statistics using the held-out B-DNA windows, then apply Benjamini-Hochberg multiple-testing correction at fdr_level=0.2 to call significant non-B windows. Write the novelty-score table (scores, P-values, calls) plus the trained model and diagnostic metric spreadsheets to the root-level results/ directory.

## Related tools

- **Python** (Implements Main.py, which trains the GoFAE-DND autoencoder, computes novelty statistics and empirical-null P-values, and applies Benjamini-Hochberg FDR) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Numerical computation of reconstruction errors, robust Mahalanobis distances, and random projections)
- **pandas** (Writes the novelty-score and metric spreadsheets to the results/ directory)
- **matplotlib** (Generates the diagnostic images accompanying the results spreadsheets)

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- results/ directory contains the metric spreadsheets and diagnostic images after a run
- Reported non-B P-value calls respect the configured FDR level of 0.2 under Benjamini-Hochberg correction
- Novelty statistics are calibrated against the empirical null built from held-out B-DNA test windows, so null P-values should be approximately uniform
- Configuration matches the documented run: nonb_ratio=0.1, n_z=64, num_projections=64, epochs=25, discriminative_weight=35, lambda_alpha=0.5
- GoFAE-DND performance is compared against isolation forest, local outlier factor, and one-class SVM baselines on the same simulated_data/ inputs

## Limitations

- GoFAE-DND code availability was stated as 'will be released upon acceptance', so availability cannot be assumed a priori
- Novelty calls depend on motif windows from the non-B DNA DB, which are necessary but not sufficient indicators of non-B structures
- Experimental FAST5 data totals over 4 TB, so reproduction can be slow and storage-heavy

## Evidence

- [abstract] A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures.: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures."
- [readme] python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2: "python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2"
- [readme] The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory.: "The results including spreadsheets of metrics and diagnostic images 
are included in the root-level results/ directory."
- [methods] we can run isolation forests, local outlier factors, and one class SVMs: "Afterwards, we can run isolation forests, local outlier factors, and one class SVMs."
- [intro] GoFAE-DND code will be released upon acceptance.: "**_GoFAE-DND code will be released upon acceptance._**"
