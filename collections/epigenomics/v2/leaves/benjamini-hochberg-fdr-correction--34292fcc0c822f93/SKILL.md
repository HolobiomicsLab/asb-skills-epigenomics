---
name: benjamini-hochberg-fdr-correction
description: Use when novelty scores (reconstruction error, Mahalanobis distance,
  IF/LOF/SVM scores) have been converted to empirical P-values against a B-DNA null
  and a per-window significance decision is needed at a fixed FDR level.
when_to_use_negative:
- When a family-wise error guarantee is required instead of FDR control (BH controls
  expected false discovery proportion, not family-wise error)
- When P-values are not calibrated against an empirical B-DNA null — BH applied to
  raw novelty scores is invalid
- For supervised classification tasks (SVC, random forest, nearest neighbors, logistic
  regression, gaussian process), which do not use FDR calling in this workflow
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: GoFAE-DND
  role: Produces the empirical P-values (via Gaussian goodness-of-fit optimization)
    that Benjamini-Hochberg corrects; fdr_level=0.2 is a Main.py parameter
  repo: github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: Python
  role: Implementation language for Main.py, the BH correction, and the novelty detector
    scripts
- name: numpy
  role: Numerical computation of P-values and BH adjustment over window arrays
- name: pandas
  role: Writing the novelty-call and comparison spreadsheets to results/
- name: conda
  role: Environment management via gofaednd_env.yml
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:d5080d07508614a46ae6089f7dec42db8ef55e6fdd51da618b0da5b03c06f971
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GoFAE-DND
  - Python
  - conda
  - h5py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# benjamini-hochberg-fdr-correction

## Summary

Apply Benjamini-Hochberg multiple-testing correction at a configured FDR level (fdr_level=0.2 in the GoFAE-DND workflow) to convert empirical-null P-values of novelty statistics into significant non-B DNA calls, controlling the false discovery rate across windows.

## When to use

Use when novelty scores (reconstruction error, Mahalanobis distance, IF/LOF/SVM scores) have been converted to empirical P-values against a B-DNA null and a per-window significance decision is needed at a fixed FDR level.

## When NOT to use

- When a family-wise error guarantee is required instead of FDR control (BH controls expected false discovery proportion, not family-wise error)
- When P-values are not calibrated against an empirical B-DNA null — BH applied to raw novelty scores is invalid
- For supervised classification tasks (SVC, random forest, nearest neighbors, logistic regression, gaussian process), which do not use FDR calling in this workflow

## Inputs

- Novelty scores or test statistics for each translocation-time window (reconstruction error, robust Mahalanobis distance, or IF/LOF/one-class-SVM scores)
- Empirical null distribution of statistics from held-out B-DNA windows (configured via nonb_ratio=0.1)
- FDR level parameter (fdr_level=0.2) passed to Main.py

## Outputs

- Per-window novelty-score table with scores, P-values, and BH-adjusted significant/non-significant calls
- Counts of significant non-B novelties per method and per non-B type (A-phased repeats, Z-DNA, mirror/direct repeats, G-quadruplexes, short tandem repeats) written to the root-level results/ directory
- Spreadsheets of metrics and diagnostic images in results/

## How to apply

Train the GoFAE-DND autoencoder (n_z=64, 64 random projections, 25 epochs, discriminative_weight=35, lambda_alpha=0.5, nonb_ratio=0.1) so that the discriminative loss makes non-B windows poorly reconstructed and Gaussian goodness-of-fit (Shapiro-Wilk) optimization yields P-values for each test window. Calibrate empirical P-values for reconstruction-error and robust Mahalanobis-distance novelty statistics against the held-out B-DNA test distribution. Then run Benjamini-Hochberg at the configured fdr_level (0.2) to call significant non-B windows; the same P-value + BH pipeline is applied per novelty-detection baseline (isolation forest, local outlier factor, one-class SVM) for fair comparison. When comparing methods on the central 50 bp experimental NA12878 windows, apply BH per method and per non-B type before tabulating call counts in the results/ spreadsheet.

## Related tools

- **GoFAE-DND** (Produces the empirical P-values (via Gaussian goodness-of-fit optimization) that Benjamini-Hochberg corrects; fdr_level=0.2 is a Main.py parameter) — github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **Python** (Implementation language for Main.py, the BH correction, and the novelty detector scripts)
- **numpy** (Numerical computation of P-values and BH adjustment over window arrays)
- **pandas** (Writing the novelty-call and comparison spreadsheets to results/)
- **conda** (Environment management via gofaednd_env.yml)

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- The results/ directory contains spreadsheets of metrics and diagnostic images, and the novelty-score table includes scores, P-values, and calls
- The fraction of false discoveries among calls is controlled at the configured fdr_level (0.2) under the empirical B-DNA null
- At FDR alpha=0.2 on central 50 bp experimental windows, SVM and GoFAE-DND call the most novelties, with GoFAE-DND strongest on A-phased repeats and Z-DNA and all methods weak on mirror and direct repeats (Supp. Table S5 pattern)
- BH is applied with the same fdr_level across GoFAE-DND and the isolation forest, local outlier factor, and one-class SVM baselines for a fair comparison

## Limitations

- GoFAE-DND code release was pending acceptance, so the full P-value/BH pipeline may need to be reimplemented from the paper's description
- All methods perform poorly on mirror and direct repeats, so FDR-controlled calls for these non-B types are expected to be few
- Raw ONT FAST5 data for the experimental workflow total over 4TB, making full reproduction slow

## Evidence

- [abstract] A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures.: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures."
- [readme] --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5: "--nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5"
- [readme] The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory.: "The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory."
