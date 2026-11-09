---
name: cross-method-network-comparison
description: Use when you need to validate a network inference tool against a baseline
  method on the same time-series dataset, comparing recovery of literature/ChIP-derived
  regulons against random regulator profiles.
when_to_use_negative:
- The data are not time-course expression measurements — Genexpi's ODE model requires
  expression dynamics over time
- Applying Genexpi to eukaryotic data expecting fully validated results — the article
  states it was designed for bacterial regulons and should be used with caution outside
  that scope
- Treating the resulting network as complete regulatory truth — time-series data cannot
  in principle provide complete interaction information, so results must be used alongside
  other sources
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Genexpi
  role: Primary ODE-based inference method; run via simulated annealing on GPU (OpenCL)
    with 128 annealing runs per gene–regulator candidate set
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: ARACNE (TD-Aracne)
  role: Baseline/co-variation method run on the same smoothed time-course data for
    the cross-method comparison
- name: R
  role: Used to preprocess (B-spline smoothing of expression profiles) and to reproduce
    results via the attached R notebook
  repo: http://github.com/cas-bioinf/genexpi/wiki/
- name: CyGenexpi
  role: Cytoscape plugin integrating Genexpi for interactive regulon discovery and
    inspection
  repo: http://github.com/cas-bioinf/genexpi/wiki/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2138-x
    title: 'Genexpi: a toolset for identifying regulons and validating gene regulatory
      networks using time-course expression data'
  source_content_hash: sha256:6ee99f168018c8176e6738ba1910a2dc447872197d4ce150799b48b727b8362c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ARACNE
  - ARACNE (TD-Aracne)
  - CyGenexpi
  - Genexpi
  - R
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cross-method-network-comparison

## Summary

Benchmark gene regulatory network inference by running two methods (e.g. Genexpi's ODE-based inference and TD-Aracne) on the same smoothed time-course data and literature-derived regulon variants, then tabulating recovered targets against published regulons and random-regulator baselines.

## When to use

Use when you need to validate a network inference tool against a baseline method on the same time-series dataset, comparing recovery of literature/ChIP-derived regulons against random regulator profiles.

## When NOT to use

- The data are not time-course expression measurements — Genexpi's ODE model requires expression dynamics over time
- Applying Genexpi to eukaryotic data expecting fully validated results — the article states it was designed for bacterial regulons and should be used with caution outside that scope
- Treating the resulting network as complete regulatory truth — time-series data cannot in principle provide complete interaction information, so results must be used alongside other sources

## Inputs

- Expression time series from GEO (e.g. GSE44415 for Streptomyces coelicolor SigR)
- Literature-derived regulon gene lists (e.g. SigR regulon from Kallifidas et al. ChIP-chip and Kim et al. knockout studies)
- Random regulator profiles as negative-control baselines
- B-spline-smoothed time-course matrices shared across methods

## Outputs

- Comparison table with one row per method-per-regulon-variant showing recovered targets (e.g. sigr_regulon_recovery_comparison.csv)
- Per-method lists of recovered regulon targets
- Genexpi fitted regulations passing the fit-quality threshold (default 0.8)

## How to apply

Obtain the expression time series (e.g. GSE44415) and the candidate regulon gene lists from literature sources (e.g. SigR regulon of Streptomyces coelicolor from Kallifidas et al. ChIP-chip and Kim et al. knockout studies). Smooth expression profiles in R by linear regression on a B-spline basis with degrees of freedom approximately half the number of measurement points, so both methods consume identical input. Run Genexpi's ODE inference via 128 simulated-annealing runs per gene–regulator set (OpenCL, regularization term r(beta) with c ≈ n_time_points/10), keeping fits with fit quality above the default 0.8 threshold, and run TD-Aracne on the same smoothed data. For each regulon variant and each method, count and list recovered targets against the published gene sets, then assemble one comparison row per method-per-regulon-variant (e.g. saved as sigr_regulon_recovery_comparison.csv). The rationale is that a method that captures real regulatory dynamics should distinguish literature-derived regulons from random profiles, whereas a purely co-variation-based method should barely differentiate the two.

## Related tools

- **Genexpi** (Primary ODE-based inference method; run via simulated annealing on GPU (OpenCL) with 128 annealing runs per gene–regulator candidate set) — http://github.com/cas-bioinf/genexpi/wiki/
- **ARACNE (TD-Aracne)** (Baseline/co-variation method run on the same smoothed time-course data for the cross-method comparison)
- **R** (Used to preprocess (B-spline smoothing of expression profiles) and to reproduce results via the attached R notebook) — http://github.com/cas-bioinf/genexpi/wiki/
- **CyGenexpi** (Cytoscape plugin integrating Genexpi for interactive regulon discovery and inspection) — http://github.com/cas-bioinf/genexpi/wiki/

## Examples

```
library(splines); smooth_fit <- lm(expr ~ bs(time, df = floor(n_points/2)))
```

## Evaluation signals

- Genexpi recovers substantially more SigR targets from literature-derived regulons than from random regulons on GSE44415
- TD-Aracne's recovery is barely distinguishable between literature-based and random regulons — the expected contrast between the two methods
- Genexpi retains only fitted regulations whose fit quality exceeds the default 0.8 threshold (proportion of time points within the error margin)
- The comparison CSV contains exactly one row per method-per-regulon-variant (2 SigR regulon variants × 2 methods)
- Genexpi should be 'rarely misled by random profiles' across both regulon variants, per the paper's reported finding

## Limitations

- Genexpi masks uncertainty in the data and uses multiple hard thresholds
- Genexpi was designed for bacterial regulons; applicability to eukaryotic data is not fully validated and requires caution
- Results are incomplete on their own — time-series expression data cannot in principle provide complete regulatory information, so the tool is best used as one of multiple sources of insight
- On this benchmark, TD-Aracne mostly found little difference between literature-based and random regulons, limiting its utility as a validating signal

## Evidence

- [methods] Simulated annealing parameters: "Genexpi minimizes eq. 2 by simulated annealing. For each gene and candidate regulator set we execute 128 annealing runs"
- [methods] Fit quality definition: "Fit quality is then the proportion of time points where the fitted profile is within the error margin of the measured profile."
- [results] Benchmark datasets: "Both versions of the SigR regulon were combined with the GSE44415 expression time series [17]."
- [results] Tool limitation: "Most notably, Genexpi masks uncertainty in the data and uses multiple hard thresholds."
- [readme] Project documentation location: "Detailed information can be found on the project's wiki: https://github.com/cas-bioinf/genexpi/wiki"
