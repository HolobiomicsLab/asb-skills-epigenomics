---
name: aic-likelihood-based-model-selection
description: Decision support for choosing the number of sub-compartments (N_subcompartment)
  when running CscoreTool-M on Hi-C data, where no prior annotation fixes the compartment
  count.
when_to_use_negative:
- The compartment count is already fixed by prior biological annotation or a published
  sub-compartment assignment — no model selection sweep is needed
- Input Hi-C data is single-compartment analysis only (A/B compartments via PCA, e.g.
  HOMER runHiCpca.pl), since AIC selection applies to the multi-sub-compartment model
- The tool being tuned is not a likelihood-based probabilistic model, because AIC
  = 2k − 2lnL requires a maximized log-likelihood from each candidate fit
tools:
- name: CscoreTool-M
  role: Probabilistic-model-based tool whose runs at each N_subcompartment value supply
    the maximum log-likelihood used in the AIC = 2k − 2lnL calculation; invoked with
    Rabl.bed, summary file, prefix, N_subcompartment, N_Rablwindow, N_session, and
    optional Blacklist.bed.
  repo: https://github.com/scoutzxb/CscoreTool-M
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad314
    title: CscoreTool-M infers 3D sub-compartment probabilities within cell population
  source_content_hash: sha256:819fa8888ecad81a544dcf1eaed5322f8605e44c50863deecffc43be7def1e63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scoutzxb/CscoreTool-M
  tools:
  - CscoreTool-M
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# aic-likelihood-based-model-selection

## Summary

Selects the statistically optimal number of sub-compartments for CscoreTool-M by sweeping the N_subcompartment argument, extracting the maximum log-likelihood from each run, and choosing the model with minimum AIC = 2k − 2lnL on down-sampled GM12878 Hi-C data.

## When to use

Decision support for choosing the number of sub-compartments (N_subcompartment) when running CscoreTool-M on Hi-C data, where no prior annotation fixes the compartment count.

## When NOT to use

- The compartment count is already fixed by prior biological annotation or a published sub-compartment assignment — no model selection sweep is needed
- Input Hi-C data is single-compartment analysis only (A/B compartments via PCA, e.g. HOMER runHiCpca.pl), since AIC selection applies to the multi-sub-compartment model
- The tool being tuned is not a likelihood-based probabilistic model, because AIC = 2k − 2lnL requires a maximized log-likelihood from each candidate fit

## Inputs

- Down-sampled (0.1x and 0.01x) GM12878 Hi-C contact summary file in HOMER runHiCpca.pl HiCsummary format (Test.summary-style)
- Equal-length genomic windows bed file with Rabl position (0 = centromere, 1 = telomere) in the 4th column (e.g. hg38_500k_Rabl.bed)
- Optional: ENCODE blacklist regions bed file (Blacklist.bed) to exclude copy-number-variable regions
- Sweep of candidate N_subcompartment values (e.g. k = 2..15+)

## Outputs

- aic_results.csv tabulating AIC versus number of compartments for each down-sampling depth
- Per-run XXX_cscore.txt files with compartment scores per genomic window
- Optimal N_subcompartment per dataset (e.g. 6 for the 0.01x GM12878 data at 100 kb resolution)

## How to apply

Down-sample the high-resolution GM12878 Hi-C contact data (Rao, 2014) to 0.1x and 0.01x depth to obtain summary files in the HOMER HiCsummary format accepted by CscoreTool-M. For each depth, run ./CscoreTool-M <Rabl.bed> <summary> <prefix> <N_subcompartment> <N_Rablwindow> <N_session> while sweeping the 4th argument (N_subcompartment) over candidate compartment counts; optionally pass the ENCODE Blacklist.bed as the 7th argument so regions affected by copy number variations are excluded. After each run, read the per-genomic-window scores from XXX_cscore.txt and extract the maximum log-likelihood of the fitted model. Compute AIC = 2k − 2lnL for each k (k = N_subcompartment) and tabulate AIC versus compartment number (aic_results.csv); the k with the minimum AIC is the statistically optimal compartment number. Validate by checking that the AIC curve reproduces the reported optima — a minimum at 6 compartments for the 0.01x data and a continued decrease up to >= 15 compartments for the 0.1x data (Supplementary Fig. S6).

## Related tools

- **CscoreTool-M** (Probabilistic-model-based tool whose runs at each N_subcompartment value supply the maximum log-likelihood used in the AIC = 2k − 2lnL calculation; invoked with Rabl.bed, summary file, prefix, N_subcompartment, N_Rablwindow, N_session, and optional Blacklist.bed.) — https://github.com/scoutzxb/CscoreTool-M

## Examples

```
./CscoreTool-M hg38_500k_Rabl.bed Test.summary Test_500k_2 5 20 12 hg38_blacklisted.bed
```

## Evaluation signals

- AIC = 2k − 2lnL is computed with k equal to the N_subcompartment value used in that run
- The AIC-versus-k curve for the 0.01x down-sampled GM12878 data reaches its minimum at 6 compartments
- The AIC curve for the 0.1x data shows a continued decrease up to >= 15 compartments, reproducing Supplementary Fig. S6
- Each run produces the expected XXX_cscore.txt output with per-genomic-window scores before its log-likelihood is extracted
- Results for both depths (0.1x and 0.01x) are tabulated together in aic_results.csv

## Limitations

- The optimal compartment number is depth-dependent: at higher coverage (0.1x) the AIC kept decreasing up to >= 15 compartments, so the 0.01x optimum of 6 does not transfer to deeper data
- Blacklist regions (ENCODE) should be supplied to exclude copy-number-variation-affected regions, and ExcludedInteractions.txt can exclude translocations; omitting these can bias the fit
- N_session must be chosen according to available computational resources, affecting run feasibility of the sweep

## Evidence

- [other] AIC curve reproduces the reported optima — a minimum at 6 compartments for the 0.01x data and a continued decrease up to >= 15 compartments for the 0.1x data (Supplementary Fig. S6): [evidence span withheld — anchor status: not_found]
- [readme] Usage line showing sweepable N_subcompartment argument and optional Blacklist.bed: "Usage: CscoreTool-M <windows_Rabl.bed> <input.summary> <OutputPrefix> <N_subcompartments> <N_Rablwindow> <N_session> [Blacklist.bed] [ExcludedInteractions.txt]"
- [readme] Blacklist exclusion rationale: "This is the encode blacklist file which lists the blacklist regions excluded from ChIP-seq analysis, possibly due to copy number variations."
- [readme] HiCsummary input format: "We accept the same format as the HiCsummary file format for HOMER runHiCpca.pl."
