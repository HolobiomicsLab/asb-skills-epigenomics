---
name: autoencoder-goodness-of-fit-modeling
description: Use when detecting non-B DNA structures as novelties from Oxford Nanopore
  translocation-time windows (e.g. NA12878 SRR15058166), needing per-window P-values
  with FDR control against B-DNA background.
when_to_use_negative:
- The task is supervised motif classification with labels — use the paper's classifiers
  (SVC, random forest, nearest neighbors, logistic regression, gaussian process) instead
  of novelty detection
- Input is raw ONT current signal rather than re-squiggled, base-level translocation
  times — Tombo re-squiggle processing must come first
- You need per-structure ground truth confirmation; the method predicts latent non-B
  structures and can only be validated indirectly against non-B DB motifs
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: GoFAE-DND
  role: Core autoencoder with goodness-of-fit regularization whose discriminative
    loss makes non-B windows poorly reconstructed and whose Gaussian GoF optimization
    yields P-values
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: isolation_forest.py
  role: Baseline novelty detector run on the same windows for comparison with GoFAE-DND
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: local_outlier_factor.py
  role: Baseline novelty detector run on the same windows for comparison with GoFAE-DND
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: svm_one_class.py
  role: One-class SVM baseline novelty detector; strongest competitor to GoFAE-DND
    at FDR alpha=0.2
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: Tombo
  role: Re-squiggles Albacore FAST5 output to obtain base-level current measurements
    needed to compute translocation times
  repo: https://nanoporetech.github.io/tombo/tutorials.html
- name: Albacore
  role: Basecalls raw ONT current and generates the event table describing the DNA
    context in the nanopore
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:5aaa87aebe4e9621aef68534c7eadf1bd9aa9987814d8c411a793d74d059d33c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Albacore
  - GoFAE-DND
  - Python
  - Tombo
  - conda
  - h5py
  - isolation_forest.py
  - local_outlier_factor.py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  - svm_one_class.py
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# autoencoder-goodness-of-fit-modeling

## Summary

Train GoFAE-DND, an autoencoder regularized by goodness-of-fit (GoF) tests, so that a discriminative loss makes non-B DNA windows poorly reconstructed and Gaussian GoF optimization yields P-values for non-B structure detection from ONT translocation times.

## When to use

Use when detecting non-B DNA structures as novelties from Oxford Nanopore translocation-time windows (e.g. NA12878 SRR15058166), needing per-window P-values with FDR control against B-DNA background.

## When NOT to use

- The task is supervised motif classification with labels — use the paper's classifiers (SVC, random forest, nearest neighbors, logistic regression, gaussian process) instead of novelty detection
- Input is raw ONT current signal rather than re-squiggled, base-level translocation times — Tombo re-squiggle processing must come first
- You need per-structure ground truth confirmation; the method predicts latent non-B structures and can only be validated indirectly against non-B DB motifs

## Inputs

- Translocation-time windows (window_size 50 or 100) over non-B motifs (G-quadruplexes, short tandem repeats, A-phased repeats, Z-DNA, mirror/direct repeats) from the experimental NA12878 dataset
- Motif-free B-DNA control windows for the background distribution
- Simulated translocation times from simulator.py (e.g. -nb 10000 -b 1000000)

## Outputs

- Per-window empirical P-values / novelty scores for non-B structures
- Benjamini-Hochberg FDR-controlled (alpha=0.2) novelty calls per method and per non-B type
- Spreadsheets of metrics and diagnostic images in the root-level results/ directory

## How to apply

Train GoFAE-DND on translocation-time windows using the paper's hyperparameters: config=1 or 2, nonb_ratio=0.1, n_z=64, num_projections=64, epochs=25, discriminative_weight=35, lambda_alpha=0.5, and fdr_level=0.2. The discriminative loss intentionally encourages non-B windows to be poorly reconstructed, while optimizing Gaussian GoF tests makes the latent residuals amenable to P-value computation indicating non-B structures. Convert per-window scores into empirical P-values against the motif-free B-DNA background distribution, then apply Benjamini-Hochberg correction at fdr_level=0.2 to call novelties. Compare against isolation forest, local outlier factor, and one-class SVM run on the same windows; results (metric spreadsheets and diagnostic images) are written to the root-level results/ directory.

## Related tools

- **GoFAE-DND** (Core autoencoder with goodness-of-fit regularization whose discriminative loss makes non-B windows poorly reconstructed and whose Gaussian GoF optimization yields P-values) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **isolation_forest.py** (Baseline novelty detector run on the same windows for comparison with GoFAE-DND) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **local_outlier_factor.py** (Baseline novelty detector run on the same windows for comparison with GoFAE-DND) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **svm_one_class.py** (One-class SVM baseline novelty detector; strongest competitor to GoFAE-DND at FDR alpha=0.2) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **Tombo** (Re-squiggles Albacore FAST5 output to obtain base-level current measurements needed to compute translocation times) — https://nanoporetech.github.io/tombo/tutorials.html
- **Albacore** (Basecalls raw ONT current and generates the event table describing the DNA context in the nanopore)

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- At FDR alpha=0.2 on central-50bp experimental windows, SVM and GoFAE-DND should call the most non-B novelties among the four methods
- GoFAE-DND should yield the most predictions for all non-B types besides G4, performing uniquely well on A-phased repeats and Z-DNA
- All four methods should perform poorly on mirror and direct repeats — failure there is expected, not a bug
- Hyperparameters must match the paper: nonb_ratio=0.1, n_z=64, num_projections=64, epochs=25, discriminative_weight=35, lambda_alpha=0.5
- Output spreadsheets of metrics and diagnostic images must appear in the root-level results/ directory

## Limitations

- Mirror and direct repeats are poorly detected by every novelty method, including GoFAE-DND
- The discriminative loss requires training windows that mix B-DNA controls with non-B windows (nonb_ratio=0.1); purely unlabeled use of the model is not described
- Downloading the NA12878 FAST5 files is slow — the FAST5 data total over 4TB

## Evidence

- [abstract] We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer.: "We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer."
- [abstract] A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures.: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures."
- [readme] The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory.: "The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory."
- [methods] we can run isolation forests, local outlier factors, and one class SVMs: "Afterwards, we can run isolation forests, local outlier factors, and one class SVMs."
