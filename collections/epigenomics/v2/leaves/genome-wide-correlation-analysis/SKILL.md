---
name: genome-wide-correlation-analysis
description: Use when you have CscoreTool-M *_cscore.txt outputs and a reference set
  of compartment/sub-compartment scores for the same 500kb hg38 windows and need a
  genome-wide Pearson correlation to judge inference accuracy or depth robustness.
when_to_use_negative:
- Input is raw Hi-C reads or an unprocessed HiCsummary file with no compartment scores
  yet inferred — run CscoreTool-M first before any correlation assessment
- Comparing scores across differently sized or non-matching genomic window sets (e.g.
  Rabl-positioned 500kb windows vs. another binning) where window-to-window correspondence
  is undefined
- Evaluating sub-compartment categorical assignments rather than continuous compartment
  scores — the correlation applies to score values, not label concordance
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3922
tools:
- name: CscoreTool-M
  role: Infer sub-compartment scores per genomic window (producing *_cscore.txt files)
    that are then correlated genome-wide against the reference model scores
  repo: https://github.com/scoutzxb/CscoreTool-M
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:1c7839d0321dfd391335d138a835bd1ee17922b324d62f0479d5316f4e8bc0f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- genome-wide-pearson-correlation-analysis
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# genome-wide-correlation-analysis

## Summary

Quantifies agreement between CscoreTool-M re-inferred sub-compartment scores and a reference 5-compartment model by computing genome-wide Pearson correlations across genomic windows, e.g. across simulated Hi-C datasets at 1x, 0.1x, and 0.01x sequencing depth.

## When to use

Use when you have CscoreTool-M *_cscore.txt outputs and a reference set of compartment/sub-compartment scores for the same 500kb hg38 windows and need a genome-wide Pearson correlation to judge inference accuracy or depth robustness.

## When NOT to use

- Input is raw Hi-C reads or an unprocessed HiCsummary file with no compartment scores yet inferred — run CscoreTool-M first before any correlation assessment
- Comparing scores across differently sized or non-matching genomic window sets (e.g. Rabl-positioned 500kb windows vs. another binning) where window-to-window correspondence is undefined
- Evaluating sub-compartment categorical assignments rather than continuous compartment scores — the correlation applies to score values, not label concordance

## Inputs

- CscoreTool-M *_cscore.txt files (Cscore estimated for each genomic window) from each simulated or real Hi-C dataset
- Original 5-compartment model compartment scores for the same hg38 500kb windows
- Simulated trans-interaction summary files at 1x, 0.1x, and 0.01x depth (HOMER runHiCpca.pl HiCsummary format) as inputs to CscoreTool-M

## Outputs

- Genome-wide Pearson correlation coefficient between re-inferred and reference sub-compartment scores at each sequencing depth (1x, 0.1x, 0.01x)
- Depth-robustness comparison showing whether correlations stay above 0.9

## How to apply

First run CscoreTool-M at each sequencing depth using the hg38 500kb Rabl bed file, the simulated or real HiCsummary-format input, and parameters N_subcompartment=5, N_Rablwindow=20, N_session=12, producing a *_cscore.txt per depth. Parse each *_cscore.txt to extract the re-inferred compartment score for every genomic window. Then compute genome-wide Pearson correlations between the re-inferred scores and the original 5-compartment model scores for each depth. In the simulation-based validation on GM12878-template data (Rao et al. 2014, GSE63525), correlations weaken at lower depth but must remain above 0.9 even on the 0.01x simulated dataset. Note the *_cscore.bedgraph outputs contain c-scores normalized to sum to 1 per genomic region, which is the representation used for visualization rather than the raw scores used for correlation.

## Related tools

- **CscoreTool-M** (Infer sub-compartment scores per genomic window (producing *_cscore.txt files) that are then correlated genome-wide against the reference model scores) — https://github.com/scoutzxb/CscoreTool-M

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed sim.summary sim_1x 5 20 12
```

## Evaluation signals

- Every genome-wide Pearson correlation between re-inferred and original 5-compartment model scores exceeds 0.9, including at the 0.01x simulated depth
- Correlations are computed over the same genomic windows defined in the hg38 500kb Rabl bed file used for CscoreTool-M runs
- Each depth (1x, 0.1x, 0.01x) produces a *_cscore.txt with a score per genomic window before correlation is attempted
- Correlation weakens monotonically (or near-monotonically) with decreasing sequencing depth but stays above the 0.9 threshold

## Limitations

- Correlations are expected to weaken at lower sequencing depth, so 0.9 is a floor validated on simulated data rather than an absolute guarantee for arbitrary inputs
- CscoreTool-M is compiled/executed on Linux (executable built on CentOS); if the prebuilt binary fails, compilation with g++ -fopenmp -O3 is required before scores can be generated
- Regions with no reads are omitted from the normalized *_cscore.bedgraph output, so score coverage is not uniform across the genome

## Evidence

- [other] In the simulation test, the inferred compartment scores correlate genome-wide with the model parameters, and although correlations weaken at lower depth, they remain higher than 0.9 even on the 0.01x simulated dataset.: [evidence span withheld — anchor status: not_found]
- [abstract] The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment.: "The compartment scores inferred using CscoreTool-M represents the probability of a genomic region locating in a specific sub-compartment"
- [readme] Cscore estimated for each genomic window output description from README.: "XXX_cscore.txt This is the Cscore estimated for each genomic window."
- [readme] CscoreTool-M run example with parameters from README.: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12"
- [methods] Normalized bedgraph visualization output description.: "XXX_cscore.bedgraph This is the bedgraph file made for visualization. Ths c-scores are normalized to add up to 1 for each genomic region."
