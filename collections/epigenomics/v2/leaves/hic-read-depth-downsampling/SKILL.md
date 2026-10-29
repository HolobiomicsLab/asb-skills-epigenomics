---
name: hic-read-depth-downsampling
description: Use when you need to determine how many Hi-C sub-compartments are statistically
  supported at reduced sequencing depth, or to test the robustness of sub-compartment
  inference to read-depth reduction by sweeping N_subcompartment and comparing AIC
  minima.
when_to_use_negative:
- Input Hi-C data is already low-depth; down-sampling further would leave too few
  interactions for stable sub-compartment inference
- The goal is to call a fixed number of sub-compartments rather than to statistically
  select the optimal number via likelihood/AIC
- The analysis targets a method other than CscoreTool-M whose log-likelihood over
  N_subcompartment is not exposed, since AIC = 2k - 2lnL cannot be computed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0622
tools:
- name: CscoreTool-M
  role: Probabilistic-modeling tool re-run on down-sampled Hi-C summary files at each
    candidate N_subcompartment to obtain maximum log-likelihoods for AIC computation
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Used to compile CscoreTool-M from source (CscoreTool-M.cpp, twister.cpp with
    -fopenmp -O3) if the prebuilt executable fails
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:1e3c1e72712618586e78c53ccc98b55da1264ac66e6b2cf81f4e097cfe1472ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  - g++
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hic-read-depth-downsampling

## Summary

Down-sample high-resolution Hi-C contact data (e.g. GM12878 from Rao 2014) to reduced read depths such as 0.1x and 0.01x, then re-run CscoreTool-M across candidate sub-compartment counts and select the statistically optimal number via Akaike Information Criterion (AIC = 2k - 2lnL).

## When to use

Use when you need to determine how many Hi-C sub-compartments are statistically supported at reduced sequencing depth, or to test the robustness of sub-compartment inference to read-depth reduction by sweeping N_subcompartment and comparing AIC minima.

## When NOT to use

- Input Hi-C data is already low-depth; down-sampling further would leave too few interactions for stable sub-compartment inference
- The goal is to call a fixed number of sub-compartments rather than to statistically select the optimal number via likelihood/AIC
- The analysis targets a method other than CscoreTool-M whose log-likelihood over N_subcompartment is not exposed, since AIC = 2k - 2lnL cannot be computed

## Inputs

- High-resolution GM12878 Hi-C contact data (Rao, 2014) to be down-sampled to 0.1x and 0.01x depth
- Down-sampled summary file in HOMER HiCsummary format (input.summary)
- Equal-length genomic windows bed file with Rabl position in the fourth column (e.g. hg38_500k_Rabl.bed)
- ENCODE blacklist regions bed file (optional, as Blacklist.bed 7th argument)

## Outputs

- Per-genomic-window sub-compartment scores (XXX_cscore.txt) for each compartment count and depth
- AIC versus number of compartments table for both depths (e.g. aic_results.csv)
- Identified optimal sub-compartment number per depth (minimum AIC)

## How to apply

Randomly down-sample the high-resolution GM12878 Hi-C contact data (Rao 2014) to 0.1x and 0.01x depth, formatting the results as a HOMER-style HiCsummary (input.summary) file. For each depth, run CscoreTool-M ('./CscoreTool-M <Rabl.bed> <summary> <prefix> <N_subcompartment> <N_Rablwindow> <N_session>') while sweeping the 4th argument over candidate compartment counts; optionally pass the ENCODE Blacklist.bed as the 7th argument so copy-number-affected regions are excluded. After each run, collect per-window scores from the XXX_cscore.txt output and extract the maximum log-likelihood, computing AIC = 2k - 2lnL where k is the N_subcompartment value used. Tabulate AIC versus compartment count for both depths (e.g. aic_results.csv) and pick the compartment number at the AIC minimum per depth.

## Related tools

- **CscoreTool-M** (Probabilistic-modeling tool re-run on down-sampled Hi-C summary files at each candidate N_subcompartment to obtain maximum log-likelihoods for AIC computation) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Used to compile CscoreTool-M from source (CscoreTool-M.cpp, twister.cpp with -fopenmp -O3) if the prebuilt executable fails)

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_2 5 20 12 hg38_blacklisted.bed
```

## Evaluation signals

- For the 0.01x down-sampled GM12878 data at 100 kb resolution, the AIC curve reaches a minimum at 6 compartments
- For the 0.1x down-sampled data, the AIC continues to decrease up to >= 15 compartments, matching Supplementary Fig. S6
- Each run produces the three expected CscoreTool-M outputs: XXX_cscore.txt, XXX_cscore.bedgraph, and XXX_rabl.txt
- AIC values are monotonically computed as 2k - 2lnL with k equal to the N_subcompartment argument used in each run
- AIC results are tabulated for both down-sampling depths (0.1x and 0.01x) in a single results file (aic_results.csv)

## Limitations

- The reported AIC optima are depth-specific: 6 compartments for 0.01x data versus >= 15 for 0.1x, so the optimal compartment number cannot be transferred between sequencing depths
- Down-sampling and parameter sweeps depend on computational resources; N_session must be chosen based on available resources
- Copy-number-affected blacklist regions can distort scores unless ENCODE Blacklist.bed is supplied as the 7th argument

## Evidence

- [supplementary] AIC minimum at 6 compartments for 0.01x data and continued decrease up to >= 15 compartments for the 0.1x data (Supplementary Fig. S6): [evidence span withheld — anchor status: not_found]
- [methods] Extract maximum log-likelihood and compute AIC = 2k - 2lnL with k = N_subcompartment: [evidence span withheld — anchor status: not_found]
- [readme] README documents CscoreTool-M usage and argument order including optional Blacklist.bed as 7th argument: "Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]"
- [readme] README describes blacklist exclusion rationale due to copy number variations: "This is the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations."
- [intro] Test summary input is 0.5% randomly selected reads from the high-resolution GM12878 Hi-C dataset: "This is 0.5% randomly selected reads in chr1 from the High-resolution GM12878 cell Hi-C dataset (Rao, 2014)."
