---
name: sequencing-depth-downsampling
description: Use when validating that a Hi-C compartment/sub-compartment inference
  method recovers stable scores at shallow sequencing depths, by simulating trans-interaction
  summary files at multiple downsampled depths (e.g. 1x, 0.1x, 0.01x).
when_to_use_negative:
- Input is already a low-depth experimental Hi-C dataset rather than a high-resolution
  reference from which downsampled simulations are derived
- The analysis goal is compartment inference on a single full-depth dataset rather
  than robustness/validation testing across depths
- Downsampling would be applied directly to the interaction summary without preserving
  the HOMER HiCsummary format required by CscoreTool-M
tools:
- name: CscoreTool-M
  role: Re-infers sub-compartment scores from each downsampled summary file using
    hg38 500kb Rabl windows with N_subcompartment=5, N_Rablwindow=20, N_session=12
  repo: https://github.com/scoutzxb/CscoreTool-M
- name: g++
  role: Fallback compiler to build CscoreTool-M from CscoreTool-M.cpp, twister.h and
    twister.cpp if the precompiled executable fails
  repo: https://github.com/scoutzxb/CscoreTool-M
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:6f32e1a945ce2782ff7caf16a7a199f0a25cb389bdd87af2a85eda9a5d5c1833
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool
  - CscoreTool-M
  - g++
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# sequencing-depth-downsampling

## Summary

Simulate reduced-depth Hi-C datasets by downsampling reads (e.g. 0.5%-selected reads from the Rao 2014 GM12878 high-resolution dataset) to test whether CscoreTool-M sub-compartment inference is robust to sequencing depth.

## When to use

Use when validating that a Hi-C compartment/sub-compartment inference method recovers stable scores at shallow sequencing depths, by simulating trans-interaction summary files at multiple downsampled depths (e.g. 1x, 0.1x, 0.01x).

## When NOT to use

- Input is already a low-depth experimental Hi-C dataset rather than a high-resolution reference from which downsampled simulations are derived
- The analysis goal is compartment inference on a single full-depth dataset rather than robustness/validation testing across depths
- Downsampling would be applied directly to the interaction summary without preserving the HOMER HiCsummary format required by CscoreTool-M

## Inputs

- High-resolution GM12878 Hi-C dataset (Rao, 2014; GEO GSE63525) reads for downsampling
- 5-compartment reference model used to simulate trans interactions
- Simulated Hi-C interaction summary files in HOMER HiCsummary format (1x, 0.1x, 0.01x depth)
- hg38 500kb Rabl windows bed file (equal-length windows with Rabl position column)

## Outputs

- Downsampled trans-interaction summary files at 1x, 0.1x, and 0.01x sequencing depth
- Per-depth CscoreTool-M outputs: *_cscore.txt (per-window sub-compartment scores), *_cscore.bedgraph, *_rabl.txt
- Genome-wide Pearson correlations between re-inferred scores and original model scores per depth

## How to apply

Start from a high-resolution reference Hi-C dataset (GM12878, Rao et al. 2014, GEO GSE63525) and build the multi-compartment model from which interactions are simulated. Generate trans-interaction summary files in HOMER HiCsummary format at each depth (1x, 0.1x, 0.01x) by randomly downsampling reads (e.g. 0.5%-selected reads as in the package test data from chr1). Run CscoreTool-M per depth with the hg38 500kb Rabl bed file, N_subcompartment = 5, N_Rablwindow = 20, N_session = 12, parsing each *_cscore.txt for re-inferred per-window scores. Validate by computing genome-wide Pearson correlations between re-inferred scores and the original 5-compartment model scores; every correlation must exceed 0.9, even at 0.01x depth.

## Related tools

- **CscoreTool-M** (Re-infers sub-compartment scores from each downsampled summary file using hg38 500kb Rabl windows with N_subcompartment=5, N_Rablwindow=20, N_session=12) — https://github.com/scoutzxb/CscoreTool-M
- **g++** (Fallback compiler to build CscoreTool-M from CscoreTool-M.cpp, twister.h and twister.cpp if the precompiled executable fails) — https://github.com/scoutzxb/CscoreTool-M

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed sim.summary sim_1x 5 20 12
```

## Evaluation signals

- Genome-wide Pearson correlation between re-inferred scores and the original 5-compartment model scores exceeds 0.9 at every depth tested
- Correlation remains above 0.9 even on the 0.01x simulated dataset (correlations weaken with depth but stay high)
- Each *_cscore.txt contains a re-inferred score for each genomic window at each depth
- cscore.bedgraph c-scores are normalized to add up to 1 per genomic region, with no-read regions absent
- CscoreTool-M runs with the expected 6 core arguments (bed, summary, prefix, 5, 20, 12) and optional Blacklist.bed / ExcludedInteractions.txt

## Limitations

- Correlations between inferred and model scores weaken as sequencing depth decreases, so very shallow data increases uncertainty
- Precompiled executable was built on CentOS and may not run on other Linux systems; recompilation with g++ -fopenmp -O3 may be needed
- N_session choice depends on available computational resources
- Optional filter files (Blacklist.bed, ExcludedInteractions.txt) alter expectations, not just remove interactions; omitting them changes results

## Evidence

- [intro] Test data derived from 0.5% selected GM12878 reads: "This is 0.5% randomly selected reads in chr1 from the High-resolution GM12878 cell Hi-C dataset (Rao, 2014)."
- [other] Correlation threshold at low depth: [evidence span withheld — anchor status: not_found]
- [methods] Representative CscoreTool-M invocation parameters: "./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_1 5 20 12"
- [readme] Normalization check for output bedgraph: "Ths c-scores are normalized to add up to 1 for each genomic region. Regions with no reads are not shown."
- [readme] Fallback compilation command: "g++ CscoreTool-M.cpp twister.cpp -fopenmp -O3 -o CscoreTool-M"
