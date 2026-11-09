---
name: translocation-time-quantile-analysis
description: Use when nanopore (ONT) per-base translocation-time data and non-B DNA
  DB motif annotations (e.g. hg38) are available and the goal is to test whether 100
  bp non-B motif windows deviate in TT quantiles from motif-free B-DNA controls, as
  in Fig. 4.
when_to_use_negative:
- The data are simulated translocation times rather than the experimental dataset
  — the paper's separate simulated pipeline (simulator.py + GoFAE-DND) is the intended
  path for simulated windows.
- The goal is non-B structure prediction/classification via GoFAE-DND or novelty detectors
  (isolation forest, LOF, one-class SVM) rather than descriptive quantile-deviation
  analysis of windows.
- Only raw FAST5 current signal is available and per-base translocation times have
  not yet been computed via Albacore basecalling and Tombo re-squiggle.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: pandas
  role: Load the per-base translocation-time dataset and aggregate TT quantiles per
    window per non-B type
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: numpy
  role: Compute the 0.05–0.95 quantiles of translocation times and deviation statistics
  repo: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- name: matplotlib
  role: Generate diagnostic plots of quantile deviations alongside the results CSV
- name: seaborn
  role: Generate diagnostic visualizations for the deviation analysis
- name: Tombo
  role: Upstream re-squiggling of Albacore FAST5 output to obtain base-level current
    measurements from which translocation times are derived
  repo: https://nanoporetech.github.io/tombo/tutorials.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad220
    title: Deep statistical modelling of nanopore sequencing translocation times reveals
      latent non-B DNA structures
  source_content_hash: sha256:d05507b2d06265d97f13588d1b98b8f76d15b641d2ecf20bc7a3f16c21128d54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
  tools:
  - Python
  - Tombo
  - matplotlib
  - numpy
  - pandas
  - seaborn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# translocation-time-quantile-analysis

## Summary

Reproduces the Fig. 4 analysis of the ONT non-B DNA paper by computing the 0.05, 0.25, 0.5, 0.75, and 0.95 quantiles of per-base translocation times (Tombo re-squiggle events converted to seconds) within 100 bp windows centred on non-B DNA motifs and comparing them against B-DNA control windows to quantify and rank deviations by non-B structure class.

## When to use

Use when nanopore (ONT) per-base translocation-time data and non-B DNA DB motif annotations (e.g. hg38) are available and the goal is to test whether 100 bp non-B motif windows deviate in TT quantiles from motif-free B-DNA controls, as in Fig. 4.

## When NOT to use

- The data are simulated translocation times rather than the experimental dataset — the paper's separate simulated pipeline (simulator.py + GoFAE-DND) is the intended path for simulated windows.
- The goal is non-B structure prediction/classification via GoFAE-DND or novelty detectors (isolation forest, LOF, one-class SVM) rather than descriptive quantile-deviation analysis of windows.
- Only raw FAST5 current signal is available and per-base translocation times have not yet been computed via Albacore basecalling and Tombo re-squiggle.

## Inputs

- Per-base translocation-time dataset from Tombo re-squiggle events (seconds = number of current measurements / ONT sampling rate)
- non-B DNA DB hg38 motif annotations (G-quadruplex, Z-DNA, short tandem repeat, A-phased repeat, etc.)
- Non-B DNA-free B-DNA control windows overlapping high-quality reads
- Reference genome hg38.fa (used upstream for Tombo re-squiggle)

## Outputs

- CSV table in results/ with per-type TT quantiles (0.05, 0.25, 0.5, 0.75, 0.95) and deviation-from-B-DNA fields plus summary statistics
- Diagnostic plots (matplotlib/seaborn) reproducing the Fig. 4 deviation ranking

## How to apply

Load the extracted per-base translocation-time dataset — produced by Tombo re-squiggling of Albacore-basecalled FAST5 files, where time duration in seconds is the ratio of the number of current measurements to the ONT sampling rate — with pandas. Fix windows of length window_size = 100 bp centred on each non-B DNA DB motif, extend positions to the opposite strand (Step 3 of window preparation), select B-DNA control windows from motif-free regions (Step 5) overlapping high-quality reads, and compute the translocation signal on non-overlapping windows (Step 6). For each non-B type (G-quadruplex, Z-DNA, short tandem repeat, A-phased repeat, etc.) and for B-DNA controls, compute the 0.05, 0.25, 0.5, 0.75, and 0.95 translocation-time quantiles per window with numpy/pandas, then aggregate deviations from the B-DNA control distribution by non-B structure class and apply the significance assessment used for Fig. 4. Write per-type quantile and deviation results plus summary statistics to a CSV in the root-level results/ directory, with matplotlib/seaborn diagnostic plots.

## Related tools

- **pandas** (Load the per-base translocation-time dataset and aggregate TT quantiles per window per non-B type) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **numpy** (Compute the 0.05–0.95 quantiles of translocation times and deviation statistics) — https://github.com/bayesomicslab/ONT-nonb-GoFAE-DND
- **matplotlib** (Generate diagnostic plots of quantile deviations alongside the results CSV)
- **seaborn** (Generate diagnostic visualizations for the deviation analysis)
- **Tombo** (Upstream re-squiggling of Albacore FAST5 output to obtain base-level current measurements from which translocation times are derived) — https://nanoporetech.github.io/tombo/tutorials.html

## Examples

```
tt.groupby(['nonb_type','window_id'])['translocation_time'].quantile(q=[0.05,0.25,0.5,0.75,0.95]).to_csv('results/tt_quantile_deviation.csv')
```

## Evaluation signals

- Output CSV exists in the root-level results/ directory and contains all required quantile (0.05, 0.25, 0.5, 0.75, 0.95) and deviation fields
- Reproduced deviation ranking matches the Fig. 4 pattern: A-phased repeats, Z-DNA, G-quadruplex, and short tandem repeats show the largest significant deviations from B-DNA across all TT quantiles
- All non-B DNA motif window types show statistically significant deviation from B-DNA controls (per the paper's reported finding for the experimental dataset)
- Window size is verified as 100 bp for the experimental dataset and windows are non-overlapping after Step 6 filtering
- Translocation times are in seconds and consistent with (number of current measurements / ONT sampling rate)

## Limitations

- Non-B DNA motifs are necessary but not sufficient indicators of actual non-B structures, so motif-window deviations reflect motif-associated signal, not confirmed structures
- The experimental analysis depends on FAST5 download (over 4 TB for SRR15058166) and Albacore/Tombo availability; Albacore assumes fixed block strides, which is why Tombo's base-level segmentation is needed for translocation times
- Findings are established on the NA12878 whole-genome nanopore dataset (SRR15058166) with window sizes 50 and 100; other genomes/platforms/runs are not validated here

## Evidence

- [readme] For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate.: "For each position on the Tombo-mapped reads, we compute the time duration in seconds as the ratio of the number of current measurements to the ONT sampling rate."
- [readme] Step 2: Fix windows of length x around motifs. Step 3: Extend the positions on the opposite strand.: "### Step 2: 
Fix windows of length x around motifs.

### Step 3: 
Extend the positions on the opposite strand."
- [results] datasets = ['experimental', 'simulated'] window_size = [50, 100]: "datasets = ['experimental', 'simulated']
window_size = [50, 100]"
- [abstract] Based on whole genome nanopore sequencing of NA12878, we show that there exist significant differences between the timing of DNA translocation for non-B DNA bases compared with B-DNA.: "Based on whole genome nanopore sequencing of NA12878, we show that there exist significant differences between the timing of DNA translocation for non-B DNA bases compared with B-DNA."
- [readme] The results including spreadsheets of metrics and diagnostic images are included in the root-level results/ directory.: "The results including spreadsheets of metrics and diagnostic images 
are included in the root-level results/ directory."
