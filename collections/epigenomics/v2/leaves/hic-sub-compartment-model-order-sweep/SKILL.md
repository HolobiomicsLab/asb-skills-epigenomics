---
name: hic-sub-compartment-model-order-sweep
description: Use when you must choose how many sub-compartments CscoreTool-M should
  infer for a Hi-C dataset at 100 kb resolution, e.g. testing 0.1x/0.01x down-sampled
  GM12878 contacts against candidate compartment counts.
when_to_use_negative:
- The goal is a single pre-specified sub-compartment count for a high-depth dataset
  rather than model-order selection on sparse (down-sampled) data
- The input is not Hi-C interaction data in HiCsummary format, or no equal-length
  Rabl-position windows bed file can be constructed
- The question is calling which sub-compartment a region belongs to rather than how
  many sub-compartments the model should use
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3922
tools:
- name: CscoreTool-M
  role: Probabilistic-modeling tool run once per candidate N_subcompartment to obtain
    likelihoods and per-window cscores from Hi-C summary data
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Compiles CscoreTool-M from CscoreTool-M.cpp, twister.h and twister.cpp with
    -fopenmp -O3 if the prebuilt executable does not work
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:dc2d7b196fcf204f1fe1fbff0f0472c2230f614bb6450715fa46aa5d2b332521
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3922
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# hic-sub-compartment-model-order-sweep

## Summary

Determines the statistically optimal number of Hi-C sub-compartments for CscoreTool-M by sweeping the N_subcompartment argument and selecting the model order that minimizes AIC = 2k - 2lnL on down-sampled GM12878 Hi-C data.

## When to use

Use when you must choose how many sub-compartments CscoreTool-M should infer for a Hi-C dataset at 100 kb resolution, e.g. testing 0.1x/0.01x down-sampled GM12878 contacts against candidate compartment counts.

## When NOT to use

- The goal is a single pre-specified sub-compartment count for a high-depth dataset rather than model-order selection on sparse (down-sampled) data
- The input is not Hi-C interaction data in HiCsummary format, or no equal-length Rabl-position windows bed file can be constructed
- The question is calling which sub-compartment a region belongs to rather than how many sub-compartments the model should use

## Inputs

- Hi-C summary file in HOMER runHiCpca.pl HiCsummary format (e.g. Test.summary), down-sampled to 0.1x or 0.01x depth
- Equal-length genomic windows bed file with fourth column giving Rabl position (0 = centromere, 1 = telomere), e.g. hg38_500k_Rabl.bed
- ENCODE Blacklist.bed regions excluded due to copy number variations (optional 7th argument)
- ExcludedInteractions.txt listing translocation regions to exclude (optional 8th argument)

## Outputs

- XXX_cscore.txt with Cscore (sub-compartment probability) per genomic window for each tested N_subcompartment
- Maximum log-likelihood per model order and derived AIC = 2k - 2lnL values
- aic_results.csv tabulating AIC versus number of compartments for each down-sampling depth
- Selected optimal compartment count (e.g. 6 for 0.01x GM12878 at 100 kb resolution)

## How to apply

Down-sample the high-resolution GM12878 Hi-C contact data (Rao, 2014) to 0.1x and 0.01x depth to produce summary files in the HOMER HiCsummary format accepted by CscoreTool-M. For each depth, run CscoreTool-M repeatedly with the 4th argument (N_subcompartment) swept over candidate compartment counts, keeping the Rabl bed file, N_Rablwindow (e.g. 20), and N_session (e.g. 12) fixed; optionally pass the ENCODE Blacklist.bed as the 7th argument so copy-number-affected regions are excluded. After each run, read per-window probabilities from the XXX_cscore.txt output, extract the maximum log-likelihood, and compute AIC = 2k - 2lnL where k is the N_subcompartment used. Tabulate AIC versus compartment number per depth (e.g. aic_results.csv) and pick the minimum: the expected validation is a minimum at 6 compartments for 0.01x data, while the 0.1x curve should keep decreasing up to >= 15 compartments (Supplementary Fig. S6).

## Related tools

- **CscoreTool-M** (Probabilistic-modeling tool run once per candidate N_subcompartment to obtain likelihoods and per-window cscores from Hi-C summary data) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Compiles CscoreTool-M from CscoreTool-M.cpp, twister.h and twister.cpp with -fopenmp -O3 if the prebuilt executable does not work)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_2 5 20 12 hg38_blacklisted.bed
```

## Evaluation signals

- AIC curve for the 0.01x down-sampled GM12878 dataset reaches its minimum at 6 compartments
- AIC curve for the 0.1x down-sampled data continues decreasing up to >= 15 compartments, matching Supplementary Fig. S6
- Each run produces the three expected outputs: XXX_cscore.txt, XXX_cscore.bedgraph, and XXX_rabl.txt
- Cscores in XXX_cscore.bedgraph are normalized to add up to 1 for each genomic region, with no-read regions absent
- Blacklist-supplied runs complete successfully with 7 arguments (and 8 when ExcludedInteractions.txt is given)

## Limitations

- The optimal compartment number is dataset- and depth-dependent: sparse 0.01x data supports only ~6 sub-compartments while 0.1x data supports >= 15, so optima do not transfer between depths
- Requires an equal-length windows bed file with Rabl positions (relative position between centromere and telomere), which must be prepared separately
- Prebuilt executable is compiled on CentOS; if it does not work it must be recompiled from source with g++ -fopenmp -O3

## Evidence

- [supplementary] AIC minimum at 6 compartments for 0.01x data: "the AIC reached a minimum at 6 compartments, indicating that 6 is the statistical optimal compartment number for this dataset"
- [methods] CscoreTool-M command line and arguments: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12"
- [methods] Blacklist exclusion rationale: "This is the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis"
- [readme] Purpose of the program: "This program is to do multiple sub-compartment analysis from Hi-C data."
- [readme] Input format accepted: "We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl."
