---
name: spin-glass-network-scoring
description: Use when gene-level p-values from transcriptomic and epigenomic data
  must be projected onto an interaction network to find dysregulated modules, or when
  two module-detection runs (e.g., FEM vs SMITE) need quantitative comparison of overlap
  and enrichment.
when_to_use_negative:
- When only individual loci with the most significant effects are needed rather than
  network-level module discovery — pruning-based approaches introduce the bias SMITE
  was designed to avoid.
- When no interactome (e.g., REACTOME) is available for the organism, since spinglass
  module detection requires a gene network.
- When p-value columns still contain NA or zero values, since these must be filtered/replaced
  before scoring or the score distribution and downstream KS tests are invalid.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0602
tools:
- name: SMITE
  role: Integrates expression and methylation p-values into gene scores and runs the
    spinglass module search; provides benchmark modules (SMITE-R)
  repo: https://github.com/GreallyLab/SMITE
- name: igraph
  role: Supplies the spinglass algorithm used to determine the best modules on the
    REACTOME interactome
- name: FEM (Functional Epigenetic Modules)
  role: Comparison method using the spin-glass algorithm to integrate gene expression
    and epigenetic data; run via DoFEM.bi for benchmarking
- name: BioNet
  role: Used to visualize modules using interactome data and module-building
- name: goseq
  role: Annotates resulting modules with pathways such as KEGG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:2412effb31900ceebdea6cdcda7c9cfa0a6f199105a3f85603088d872cf846bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - BioNet
  - FEM (Functional Epigenetic Modules)
  - R
  - SMITE
  - goseq
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# spin-glass-network-scoring

## Summary

Score genes from integrated expression/methylation p-values and detect network modules with igraph's spin-glass algorithm on the REACTOME interactome, then benchmark modules (e.g., FEM vs SMITE-R/SMITE-F) by gene overlap and two-sided Kolmogorov-Smirnov tests of high-scoring gene enrichment.

## When to use

Use when gene-level p-values from transcriptomic and epigenomic data must be projected onto an interaction network to find dysregulated modules, or when two module-detection runs (e.g., FEM vs SMITE) need quantitative comparison of overlap and enrichment.

## When NOT to use

- When only individual loci with the most significant effects are needed rather than network-level module discovery — pruning-based approaches introduce the bias SMITE was designed to avoid.
- When no interactome (e.g., REACTOME) is available for the organism, since spinglass module detection requires a gene network.
- When p-value columns still contain NA or zero values, since these must be filtered/replaced before scoring or the score distribution and downstream KS tests are invalid.

## Inputs

- methylation p-value table (data(methylation) from SMITEdatasets)
- curated RNA-seq expression p-value table (data(curated_expressiondata))
- REACTOME interactome as igraph object (Reactome.Symbol.Igraph.rda from the SMITE package)
- SMITE-scored annotation object (PvalueAnnotation after scorePval)
- FEM module output from DoFEM.bi for benchmarking

## Outputs

- Network modules detected by the spin-glass algorithm (SMITE and/or FEM module gene sets)
- Gene-level overlap counts between module sets (e.g., 8 SMITE-R / 23 SMITE-F overlapping genes)
- Two-sided KS-test p-values for enrichment of high-scoring genes per module set
- Fig. 8-style side-by-side module comparison plot (fem_smite_comparison.png)

## How to apply

Load the T. gondii HFF methylation data (data(methylation)) and curated RNA-seq expression (data(curated_expressiondata)) from SMITEdatasets, removing NA p-values and replacing zero p-values with the minimum non-zero value before scoring. Score genes (e.g., scorePval with weights such as methylation_promoter=0.3) and detect modules by running igraph's spinglass algorithm on the REACTOME interactome (Reactome.Symbol.Igraph.rda) with maxsize=50 and num_iterations=1000. Run the same detection on a comparison method such as FEM (DoFEM.bi) using the vignette defaults: CpG methylation averaged within 200 bp of the TSS, the first exon, then 1500 bp around the TSS. Count gene-level overlaps between module sets and apply two-sided Kolmogorov-Smirnov tests to module gene-score distributions to test enrichment for high-scoring genes. Judge success by reproducing reported module counts (7 FEM modules / 175 genes), overlap counts (8 genes vs SMITE-R, 23 vs SMITE-F), and KS p-values (SMITE-R and SMITE-F p = 0.00001; FEM p = 0.2153).

## Related tools

- **SMITE** (Integrates expression and methylation p-values into gene scores and runs the spinglass module search; provides benchmark modules (SMITE-R)) — https://github.com/GreallyLab/SMITE
- **igraph** (Supplies the spinglass algorithm used to determine the best modules on the REACTOME interactome)
- **FEM (Functional Epigenetic Modules)** (Comparison method using the spin-glass algorithm to integrate gene expression and epigenetic data; run via DoFEM.bi for benchmarking)
- **BioNet** (Used to visualize modules using interactome data and module-building)
- **goseq** (Annotates resulting modules with pathways such as KEGG)

## Examples

```
load(system.file("data","Reactome.Symbol.Igraph.rda", package="SMITE")); modules <- runSpinglass(test_annotation, network=REACTOME, maxsize=50, num_iterations=1000)
```

## Evaluation signals

- FEM module detection yields 7 modules covering 175 genes on the T. gondii HFF data.
- Gene-overlap counts match reported values: 8 overlapping genes with SMITE-R and 23 with SMITE-F.
- KS-test p-values reproduce reported enrichment: SMITE-R and SMITE-F p = 0.00001, FEM p = 0.2153.
- P-value-count bias assessment matches FEM KS p < 10^-12.
- Methylation p-value preprocessing confirms no NA rows and zeros replaced by the minimum non-zero p-value before scoring.

## Limitations

- FEM modules were not enriched for high-scoring genes (KS p = 0.2153), reflecting its paradigm of pruning to the most significant loci.
- Current integrative methods introduce bias by pruning data sets, either focusing only on the most significant loci or requiring progressively smaller pairwise intersections.
- Implementation differences matter: the Fortran re-implementation (SMITE-F) overlapped FEM in 23 genes vs 8 for the R-package run (SMITE-R).

## Evidence

- [abstract] SMITE outperforms Functional Epigenetic Modules (FEM), the current paradigm of using the spin-glass algorithm to integrate gene expression and epigenetic data: "SMITE outperforms Functional Epigenetic Modules (FEM), the current paradigm of using the spin-glass algorithm to integrate gene expression and epigenetic data"
- [methods] we use igraph's spinglass algorithm to determine the best modules: "we use igraph's spinglass algorithm to determine the best modules"
- [methods] Remove NAs from p-value column: "## Remove NAs from p-value column ##"
- [methods] Replace zeros with minimum non-zero p-value: "## Replace zeros with minimum non-zero p-value ##"
- [intro] Current methods to explore multiple coincident processes using integrated analysis introduce bias by pruning data sets: "Current methods to explore multiple coincident processes using integrated analysis introduce bias by pruning data sets, either by focusing only on a subset of loci with the most significant effects"
