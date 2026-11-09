---
name: empirical-pvalue-computation
description: Use when novelty/outlier scores (reconstruction or model-based) must
  be turned into calibrated per-window P-values and FDR-controlled non-B DNA calls
  on ONT translocation-time windows.
when_to_use_negative:
- Input data is simulated rather than experimental translocation-time windows, where
  the specific B-DNA empirical background calibration for NA12878 ONT data does not
  apply
- The goal is supervised classification (SVC, random forest, etc.) on labeled non-B
  windows rather than FDR-controlled novelty calling without labels
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: GoFAE-DND
  role: Autoencoder whose discriminative loss makes non-B windows poorly reconstructed;
    Gaussian goodness-of-fit optimization yields the P-values indicating non-B structures
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: isolation_forest.py
  role: Baseline novelty detector whose scores are converted into empirical P-values
    against the B-DNA background
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: local_outlier_factor.py
  role: Baseline novelty detector whose scores are converted into empirical P-values
    against the B-DNA background
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: svm_one_class.py
  role: One-class SVM baseline whose novelty scores are converted into empirical P-values
    against the B-DNA background
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Numerical computation for empirical P-value ranking and BH correction
- name: pandas
  role: Tabulating significant novelty counts per method per non-B type into the results
    spreadsheet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:fa845a7c636e99b7a0c5f2e6925ad7d68d78fa9355dae021c3b378f7f7ff84ae
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
  - svm_one_class.py
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# empirical-pvalue-computation

## Summary

Converts per-window novelty scores (from GoFAE-DND Gaussian goodness-of-fit optimization, isolation forest, LOF, or one-class SVM) into empirical P-values against a motif-free B-DNA background distribution, then applies Benjamini-Hochberg FDR correction at fdr_level=0.2 to call non-B DNA novelties.

## When to use

Use when novelty/outlier scores (reconstruction or model-based) must be turned into calibrated per-window P-values and FDR-controlled non-B DNA calls on ONT translocation-time windows.

## When NOT to use

- Input data is simulated rather than experimental translocation-time windows, where the specific B-DNA empirical background calibration for NA12878 ONT data does not apply
- The goal is supervised classification (SVC, random forest, etc.) on labeled non-B windows rather than FDR-controlled novelty calling without labels

## Inputs

- Per-window novelty scores from GoFAE-DND, isolation_forest.py, local_outlier_factor.py, or svm_one_class.py on experimental TT windows
- Empirical score distribution from motif-free B-DNA control windows
- Non-B motif windows (A-phased repeats, Z-DNA, mirror/direct repeats, G-quadruplexes, short tandem repeats) from the experimental dataset

## Outputs

- Empirical P-values per window per method
- Benjamini-Hochberg FDR-controlled novelty calls at alpha=0.2, per method and per non-B type
- Comparison spreadsheet of significant novelty counts written to the root-level results/ directory

## How to apply

After each detector scores the central 50 bp (or 100 bp) translocation-time windows, rank each window's novelty score against the empirical distribution of scores from motif-free B-DNA control windows to obtain an empirical P-value. For GoFAE-DND specifically, the discriminative loss (discriminative_weight=35) makes non-B windows poorly reconstructed and optimizing Gaussian goodness-of-fit tests yields P-values directly. Convert isolation forest, local outlier factor, and one-class SVM scores into empirical P-values the same way, against the B-DNA background. Then apply Benjamini-Hochberg correction with fdr_level=0.2 (lambda_alpha=0.5 in the GoFAE-DND loss) per method and per non-B type (A-phased repeats, Z-DNA, mirror/direct repeats, G-quadruplexes, short tandem repeats) to call significant novelties. Tabulate significant calls per method per non-B type and write the comparison spreadsheet to the root-level results/ directory.

## Related tools

- **GoFAE-DND** (Autoencoder whose discriminative loss makes non-B windows poorly reconstructed; Gaussian goodness-of-fit optimization yields the P-values indicating non-B structures) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **isolation_forest.py** (Baseline novelty detector whose scores are converted into empirical P-values against the B-DNA background) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **local_outlier_factor.py** (Baseline novelty detector whose scores are converted into empirical P-values against the B-DNA background) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **svm_one_class.py** (One-class SVM baseline whose novelty scores are converted into empirical P-values against the B-DNA background) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Numerical computation for empirical P-value ranking and BH correction)
- **pandas** (Tabulating significant novelty counts per method per non-B type into the results spreadsheet)

## Examples

```
python Main.py --data_type=experimental --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- At FDR alpha=0.2, SVM and GoFAE-DND generate the most novelties compared with isolation forest and LOF
- GoFAE-DND yields the most predictions for all non-B types besides G4
- GoFAE-DND performs uniquely well on A-phased repeats and Z-DNA
- All methods perform poorly on mirror and direct repeats
- The comparison table is written to the root-level results/ directory and matches Supp. Table S5 counts per method per non-B type

## Limitations

- All methods perform poorly on mirror and direct repeats, so P-value calibration does not rescue detection for these structures
- Reliable detection depends on Tombo re-squiggled base-level current measurements; computing translocation times as the ratio of current measurements to the ONT sampling rate is a prerequisite
- The comparison is specific to the central 50 bp region of experimental windows (flanking B-DNA removed); different window sizes or regions change the background distribution

## Evidence

- [abstract] A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures.: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures."
- [readme] python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5: "--nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5"
- [methods] we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
- [results] At FDR alpha=0.2, SVM and GoFAE-DND generated the most novelties, with GoFAE-DND yielding the most predictions for all non-B types besides G4: [evidence span withheld — anchor status: not_found]
