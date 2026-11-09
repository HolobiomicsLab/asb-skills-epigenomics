---
name: autoencoder-gof-regularized-scoring
description: Use when detecting non-B DNA structures from ONT translocation-time windows
  as a novelty detection problem, where only canonical B-DNA training windows and
  labeled non-B motifs (e.g., G-quadruplex, short tandem repeat) are available.
when_to_use_negative:
- When labeled non-B and B-DNA data are plentiful and a fully supervised classifier
  is preferred — the paper shows novelty detection framing is for cases where non-B
  structure labels are sparse
- When the input is raw nanopore current rather than per-base translocation times
  — the Tombo re-squiggle pipeline must run first to compute time durations
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Python
  role: Runs Main.py to train GoFAE-DND and produce novelty scores and metric spreadsheets
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Numerical backend for translocation-time matrices and novelty statistics
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: pandas
  role: Writes spreadsheets of metrics, novelty scores, P-values, and FDR calls to
    results/
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: matplotlib
  role: Renders diagnostic images accompanying the metric spreadsheets
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:9bf9ef9462939161c7888f1dc07af7117d4e85429ccf43c957d069e2e17d87e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - h5py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# autoencoder-gof-regularized-scoring

## Summary

Train the GoFAE-DND autoencoder on B-DNA nanopore translocation-time windows, using goodness-of-fit (Shapiro-Wilk) tests on the latent encoding as a regularizer so non-B DNA is poorly reconstructed, then score held-out windows by reconstruction error and robust Mahalanobis distance with empirical-null P-values.

## When to use

Use when detecting non-B DNA structures from ONT translocation-time windows as a novelty detection problem, where only canonical B-DNA training windows and labeled non-B motifs (e.g., G-quadruplex, short tandem repeat) are available.

## When NOT to use

- When labeled non-B and B-DNA data are plentiful and a fully supervised classifier is preferred — the paper shows novelty detection framing is for cases where non-B structure labels are sparse
- When the input is raw nanopore current rather than per-base translocation times — the Tombo re-squiggle pipeline must run first to compute time durations

## Inputs

- Simulated or experimental ONT translocation-time windows (e.g., simulator.py output with -nb 10000 -b 1000000 non-B/B windows)
- B-DNA training windows and held-out B-DNA windows for empirical-null calibration
- Non-B DNA test windows for a chosen motif type and non-B ratio (e.g., --nonb_ratio=0.1)

## Outputs

- Novelty-score table (scores, P-values, FDR calls per window) written to the root-level results/ directory
- Trained GoFAE-DND model
- Spreadsheets of metrics and diagnostic images in results/

## How to apply

Split translocation-time windows into B-DNA training, held-out B-DNA (empirical null), and non-B test sets at a configured non-B ratio. Train the autoencoder on B-DNA windows with n_z=64, 64 random projections, 25 epochs, discriminative_weight=35, and lambda_alpha=0.5, so the discriminative loss drives poor reconstruction of non-B-like inputs while a Gaussian goodness-of-fit (Shapiro-Wilk) test on the encoding regularizes the latent representation. Compute reconstruction-error and robust Mahalanobis-distance novelty statistics from the trained encoder, calibrate empirical null P-values against the held-out B-DNA windows, and apply Benjamini-Hochberg correction at fdr_level=0.2 to call significant non-B windows. The rationale is that optimizing Gaussian GoF tests enables P-value computation that indicates non-B structures without requiring labels at detection time.

## Related tools

- **Python** (Runs Main.py to train GoFAE-DND and produce novelty scores and metric spreadsheets) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Numerical backend for translocation-time matrices and novelty statistics) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **pandas** (Writes spreadsheets of metrics, novelty scores, P-values, and FDR calls to results/) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **matplotlib** (Renders diagnostic images accompanying the metric spreadsheets) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- results/ directory contains spreadsheets of metrics and diagnostic images after the run
- Reported P-value calls respect the configured Benjamini-Hochberg FDR level of 0.2
- On simulated data at the most challenging non-B ratio 0.05, GoFAE-DND achieves F1 of 0.2812 (G-quadruplex) and 0.2344 (short tandem repeat), ranking above SVM, LOF, IF, and supervised classifiers
- Novelty scores for non-B windows exceed the empirical null distribution built from held-out B-DNA test samples

## Limitations

- The scoring depends on an adequate held-out B-DNA set for empirical-null P-value calibration; a poorly matched null undermines FDR calls
- Detection relies on non-B DNA producing measurable differences in translocation timing; motifs with weak signal effects at low non-B ratios (e.g., 0.05) yield low absolute F1 scores even when ranking first
- The README notes simulation and training may be slow ('This may take a couple minutes'; FAST5 downloads exceed 4TB for experimental data)

## Evidence

- [abstract] GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer: "We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer."
- [abstract] Discriminative loss and Gaussian GoF P-values: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures."
- [readme] README invocation with all key hyperparameters: "$ python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2"
- [readme] Results location: "The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory."
