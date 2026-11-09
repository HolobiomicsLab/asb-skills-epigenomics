---
name: nanopore-translocation-time-novelty-detection
description: Use when ONT nanopore FAST5 reads (e.g. NA12878 / SRR15058166) must be
  converted to per-base translocation times and scored for non-B DNA novelty against
  motif-free B-DNA controls at an FDR-controlled level.
when_to_use_negative:
- Input is not nanopore current signal (e.g. Illumina reads or SMRT data, where sequencing
  speed is determined by polymerization rather than pore translocation)
- The goal is supervised classification with abundant labeled non-B examples at balanced
  ratios; the paper shows novelty detectors are most valuable at extreme non-B:B ratios
  (e.g. 0.05) and classifiers underperform there
- Only motif sequence information is available without signal-level data — motifs
  are necessary but not sufficient indicators of non-B structures
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: GoFAE-DND
  role: 'Core novelty detection model: autoencoder with goodness-of-fit regularizer
    producing P-values for non-B structures'
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: Albacore
  role: Basecalls raw ONT current into FASTQ/FAST5 with an event table describing
    the DNA context
- name: Tombo
  role: Re-squiggles Albacore FAST5 output against hg38 to segment raw current into
    base-level events, enabling translocation time computation
- name: numpy
  role: Numerical backend for simulator and analysis scripts
- name: pandas
  role: Tabulates metric spreadsheets and comparison tables
- name: h5py
  role: Dependency for the novelty detector scripts operating on windowed data
- name: isolation_forest.py / local_outlier_factor.py / svm_one_class.py
  role: Baseline novelty detectors (IF, LOF, one-class SVM) run on the same windows
    for benchmarking
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:ae995bd714419e4c762d4a7839efda3652ebcd42ddd81c2ead3bb891877e1e21
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
  - isolation_forest.py / local_outlier_factor.py / svm_one_class.py
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# nanopore-translocation-time-novelty-detection

## Summary

Detects non-B DNA structures (G-quadruplexes, short tandem repeats, A-phased repeats, Z-DNA, mirror/direct repeats) from Oxford Nanopore translocation times by formalizing detection as a novelty detection problem, using GoFAE-DND (a goodness-of-fit-regularized autoencoder) against Isolation Forest, LOF, and one-class SVM baselines.

## When to use

Use when ONT nanopore FAST5 reads (e.g. NA12878 / SRR15058166) must be converted to per-base translocation times and scored for non-B DNA novelty against motif-free B-DNA controls at an FDR-controlled level.

## When NOT to use

- Input is not nanopore current signal (e.g. Illumina reads or SMRT data, where sequencing speed is determined by polymerization rather than pore translocation)
- The goal is supervised classification with abundant labeled non-B examples at balanced ratios; the paper shows novelty detectors are most valuable at extreme non-B:B ratios (e.g. 0.05) and classifiers underperform there
- Only motif sequence information is available without signal-level data — motifs are necessary but not sufficient indicators of non-B structures

## Inputs

- ONT FAST5 raw current files (e.g. NA12878 run SRR15058166, >4 TB)
- Reference genome FASTA (hg38.fa)
- Motif positions from the non-B DNA DB (G-quadruplex, short tandem repeat, A-phased repeat, Z-DNA, mirror/direct repeat motifs)
- Simulated translocation-time windows from the bundled simulator (python simulator.py -nb 10000 -b 1000000)

## Outputs

- Per-window novelty P-values and Benjamini-Hochberg-adjusted non-B calls per method and per non-B type
- Spreadsheets of metrics (F1 scores per method x dataset x non-B ratio) and diagnostic images in the root-level results/ directory
- Comparison tables ranking GoFAE-DND vs IF, LOF, one-class SVM, and supervised classifiers (SVC, random forest, nearest neighbors, logistic regression, Gaussian process)

## How to apply

Basecall FAST5 files with Albacore (FLO-PRO002 flowcell, SQK-LSK109 kit, r941_450bps_linear_prom.cfg), then re-squiggle with Tombo against hg38 to obtain base-level current events; compute translocation time per position as number of current measurements divided by the ONT sampling rate. Extract motif positions from the non-B DNA DB, fix windows of length 50 or 100 bp around motifs, extend to the opposite strand, find high-quality reads overlapping the windows, identify motif-free B-DNA regions, and compute the translocation signal on non-overlapping windows. Run GoFAE-DND (python Main.py --data_type=simulated or experimental) with n_z=64, num_projections=64, epochs=25, fdr_level=0.2, discriminative_weight=35, lambda_alpha=0.5, nonb_ratio=0.1; the discriminative loss makes non-B windows poorly reconstructed and Gaussian goodness-of-fit optimization yields P-values. Convert novelty scores (including Isolation Forest, LOF, and one-class SVM baselines run via isolation_forest.py, local_outlier_factor.py, svm_one_class.py with -nb 20000 -b 200000) into empirical P-values against the B-DNA background and apply Benjamini-Hochberg correction at alpha=0.2 to call novelties.

## Related tools

- **GoFAE-DND** (Core novelty detection model: autoencoder with goodness-of-fit regularizer producing P-values for non-B structures) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **Albacore** (Basecalls raw ONT current into FASTQ/FAST5 with an event table describing the DNA context)
- **Tombo** (Re-squiggles Albacore FAST5 output against hg38 to segment raw current into base-level events, enabling translocation time computation)
- **numpy** (Numerical backend for simulator and analysis scripts)
- **pandas** (Tabulates metric spreadsheets and comparison tables)
- **h5py** (Dependency for the novelty detector scripts operating on windowed data)
- **isolation_forest.py / local_outlier_factor.py / svm_one_class.py** (Baseline novelty detectors (IF, LOF, one-class SVM) run on the same windows for benchmarking) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND

## Examples

```
python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2 --discriminative_weight=35 --lambda_alpha=0.5
```

## Evaluation signals

- On simulated G4 and STR data at non-B ratio 0.05, GoFAE-DND should attain F1 ≈ 0.2812 (G-quadruplex) and 0.2312–0.2344 (STR), ranking above SVM, LOF, then IF, and above supervised classifiers
- On experimental central-50 bp windows at FDR alpha=0.2, SVM and GoFAE-DND should call the most novelties, with GoFAE-DND strongest on A-phased repeats and Z-DNA
- All methods should perform poorly on mirror and direct repeats — a large number of calls there signals a pipeline error
- Results (metric spreadsheets, diagnostic images) should appear in the root-level results/ directory as produced by Main.py and the detector/classifier scripts
- P-values must derive from the Gaussian goodness-of-fit optimization on the B-DNA background before Benjamini-Hochberg correction at fdr_level=0.2

## Limitations

- GoFAE-DND code availability was contingent on paper acceptance ('code will be released upon acceptance'); verify the repo is populated before relying on it
- The experimental FAST5 download exceeds 4 TB, so full reproduction is slow and storage-intensive
- Motifs from the non-B DNA DB are necessary but not sufficient indicators of actual non-B structures; calls are latent-structure predictions, not direct observations
- Mirror and direct repeats are poorly detected by all methods, so absence of calls for these types is expected, not a bug

## Evidence

- [abstract] We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer.: "We formalize non-B detection as a novelty detection problem and develop the GoFAE-DND, an autoencoder that uses goodness-of-fit (GoF) tests as a regularizer."
- [abstract] A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values that indicate non-B structures.: "A discriminative loss encourages non-B DNA to be poorly reconstructed and optimizing Gaussian GoF tests allows for the computation of P-values"
- [methods] Tombo resquiggle command and translocation time computation from current measurements and sampling rate.: "we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate"
- [readme] README command running GoFAE-DND on simulated data with full hyperparameters.: "python Main.py --data_type=simulated --sim_data_path=../../simulated_data/ --config=1 --nonb_type=G_Quadruplex_Motif --nonb_ratio=0.1 --n_z=64 --num_projections=64 --epochs=25 --fdr_level=0.2"
- [readme] README note on FAST5 size cautioning reproduction cost.: "Note: this may be slow as the FAST5 files total over 4TB"
