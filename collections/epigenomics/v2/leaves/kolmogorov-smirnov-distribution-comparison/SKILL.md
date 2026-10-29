---
name: kolmogorov-smirnov-distribution-comparison
description: Use when two sets of network modules each carry per-gene significance/score
  values and the question is whether one method's modules are enriched for high-scoring
  genes relative to the other (KS p-value comparison).
when_to_use_negative:
- Module sets lack per-gene significance or score values, so no distribution comparison
  is possible.
- The goal is differential expression or module detection itself rather than comparing
  score distributions between already-detected module sets.
- Gene sets are so small that KS-test asymptotics are unreliable (e.g., single-module
  overlap of a handful of genes only).
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SMITE
  role: Provides the scored modules (SMITE-R, SMITE-F) whose gene-score distributions
    are tested for enrichment against FEM modules
  repo: https://github.com/GreallyLab/SMITE
- name: FEM (Functional Epigenetic Modules)
  role: Baseline spin-glass method whose module gene-score distribution is compared
    to SMITE's via KS tests
- name: igraph
  role: Spinglass algorithm used to detect modules on the REACTOME interactome for
    both FEM and SMITE runs being compared
- name: R
  role: Analysis environment in which the KS tests and module comparison are performed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:ab22e5701c14ee6a00670c753674cfaa892f9cb63b44c9b18a26262b8922ae5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - FEM (Functional Epigenetic Modules)
  - R
  - SMITE
  - igraph
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# kolmogorov-smirnov-distribution-comparison

## Summary

Applies two-sided Kolmogorov-Smirnov tests to compare gene-score distributions between network modules (e.g., FEM spin-glass modules vs. SMITE-R/SMITE-F modules in the T. gondii HFF dataset), quantifying whether one module-detection method enriches for high-scoring genes relative to another.

## When to use

Use when two sets of network modules each carry per-gene significance/score values and the question is whether one method's modules are enriched for high-scoring genes relative to the other (KS p-value comparison).

## When NOT to use

- Module sets lack per-gene significance or score values, so no distribution comparison is possible.
- The goal is differential expression or module detection itself rather than comparing score distributions between already-detected module sets.
- Gene sets are so small that KS-test asymptotics are unreliable (e.g., single-module overlap of a handful of genes only).

## Inputs

- SMITE module gene-score distributions (SMITE-R and SMITE-F runs on the Toxoplasma gondii HFF methylation and curated_expressiondata datasets)
- FEM module gene sets and scores (7 modules / 175 genes) from DoFEM.bi on matched expression/methylation data using REACTOME interactome
- Gene-level overlap counts between FEM and SMITE module gene sets

## Outputs

- Two-sided KS p-values per module-set comparison (SMITE-R p = 0.00001, SMITE-F p = 0.00001, FEM p = 0.2153)
- Module gene-overlap counts (8 genes SMITE-R, 23 genes SMITE-F)
- P-value-count bias assessment statistic (FEM KS p < 10^-12)
- Fig. 8-style side-by-side module comparison plot (fem_smite_comparison.png)

## How to apply

First collect the gene-level scores for each module set: FEM modules detected with the spin-glass algorithm on the REACTOME interactome via igraph (parallel to SMITE's runSpinglass call with network=REACTOME, maxsize=50, num_iterations=1000, yielding 7 modules / 175 genes), and SMITE modules from both the R-package run (SMITE-R) and the Fortran re-implementation (SMITE-F). Compute the gene-level overlap between module sets (expected: 8 genes for SMITE-R, 23 for SMITE-F). Then apply two-sided Kolmogorov-Smirnov tests to the module gene-score distributions, treating a small p-value (e.g., p = 0.00001 for SMITE-R and SMITE-F) as enrichment for high-scoring genes and a large p-value (e.g., p = 0.2153 for FEM) as no enrichment. Rationale: SMITE's significance-based scoring assigns p-values to genes and modules, so KS tests directly test whether a method's modules concentrate statistically significant members, complementing the p-value-count bias assessment (FEM KS p < 10^-12).

## Related tools

- **SMITE** (Provides the scored modules (SMITE-R, SMITE-F) whose gene-score distributions are tested for enrichment against FEM modules) — https://github.com/GreallyLab/SMITE
- **FEM (Functional Epigenetic Modules)** (Baseline spin-glass method whose module gene-score distribution is compared to SMITE's via KS tests)
- **igraph** (Spinglass algorithm used to detect modules on the REACTOME interactome for both FEM and SMITE runs being compared)
- **R** (Analysis environment in which the KS tests and module comparison are performed)

## Examples

```
ks.test(module_scores_smite_r, module_scores_fem, alternative = "two.sided")
```

## Evaluation signals

- FEM run reproduces the reported 7 modules covering 175 genes.
- Gene-overlap counts match reported values: 8 genes (SMITE-R) and 23 genes (SMITE-F).
- KS p-values match reported results: SMITE-R and SMITE-F p = 0.00001, FEM p = 0.2153.
- P-value-count bias assessment reproduces FEM KS p < 10^-12.
- Side-by-side comparison plot renders successfully and is saved as fem_smite_comparison.png.

## Limitations

- FEM modules were not enriched for high-scoring genes in this dataset, so the KS result is dataset- and scoring-dependent rather than a universal ordering of the two methods.
- Existing integrative methods introduce bias by pruning datasets (e.g., focusing only on loci with the most significant effects), which the p-value-count bias assessment (KS p < 10^-12) is intended to expose.
- Current genomic experiments are underpowered to detect genomic events comprehensively within a network, limiting the statistical power of module-level comparisons.

## Evidence

- [abstract] SMITE outperforms Functional Epigenetic Modules (FEM), the current paradigm of using the spin-glass algorithm to integrate gene expression and epigenetic data: "SMITE outperforms Functional Epigenetic Modules (FEM), the current paradigm of using the spin-glass algorithm to integrate gene expression and epigenetic data"
- [methods] we use igraph's spinglass algorithm to determine the best modules: "we use igraph's spinglass algorithm to determine the best modules"
- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [discussion] Modules implicate potentially important network members for which there may be no statistically significant evidence.: "Modules implicate potentially important network members for which there may be no statistically significant evidence."
