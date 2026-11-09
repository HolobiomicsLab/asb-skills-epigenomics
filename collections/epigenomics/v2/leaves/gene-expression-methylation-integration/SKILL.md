---
name: gene-expression-methylation-integration
description: Use when integrating matched transcriptomic and epigenomic (methylation)
  p-value data to score genes, detect dysregulated network modules, or compare module-detection
  methods such as SMITE and FEM.
when_to_use_negative:
- Data sets are independent/unpaired across genomic layers, so correlation-aware p-value
  combination is inappropriate or cannot be matched at the gene level.
- Only one data type is available (expression or methylation alone) and the goal is
  single-omics differential analysis rather than integrative module discovery.
- The input is already a module membership table; overlap and KS enrichment testing
  applies, but the SMITE scoring/module-building pipeline itself should not be re-run.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: SMITE
  role: R/Bioconductor package that annotates expression and methylation p-values,
    combines them with correlation correction, and detects network modules via spin-glass
  repo: https://github.com/GreallyLab/SMITE
- name: SMITEdatasets
  role: provides the curated methylation, expression, hg19 annotation, histone, and
    REACTOME igraph data objects used as inputs
  repo: https://github.com/GreallyLab/SMITEdatasets
- name: igraph
  role: supplies the spinglass algorithm used to determine the best modules on the
    REACTOME interactome
- name: FEM (Functional Epigenetic Modules)
  role: benchmark spin-glass method integrating gene expression and epigenetic data
    for module comparison
- name: BioNet
  role: visualizes detected modules using interactome data
- name: goseq
  role: goseq-like approach to annotate modules with pathways such as KEGG
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:776d33ad3462c6dbe3e0e947f874b96bf15e4df445fc450879f398b57a6d0412
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - BioNet
  - FEM (Functional Epigenetic Modules)
  - R
  - SMITE
  - SMITEdatasets
  - goseq
  - igraph
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gene-expression-methylation-integration

## Summary

Integrate gene-expression and DNA-methylation p-values into per-gene significance scores and network modules using SMITE, which combines p-values while accounting for correlation between non-independent values, and benchmark the resulting modules against the FEM spin-glass approach on the same T. gondii HFF infection data.

## When to use

Use when integrating matched transcriptomic and epigenomic (methylation) p-value data to score genes, detect dysregulated network modules, or compare module-detection methods such as SMITE and FEM.

## When NOT to use

- Data sets are independent/unpaired across genomic layers, so correlation-aware p-value combination is inappropriate or cannot be matched at the gene level.
- Only one data type is available (expression or methylation alone) and the goal is single-omics differential analysis rather than integrative module discovery.
- The input is already a module membership table; overlap and KS enrichment testing applies, but the SMITE scoring/module-building pipeline itself should not be re-run.

## Inputs

- methylation data (data(methylation)) with gene-level CpG p-values
- curated RNA-seq expression data (data(curated_expressiondata))
- REACTOME interactome as igraph object (Reactome.Symbol.Igraph.rda)
- hg19 gene annotation BED (data(hg19_genes_bed))
- histone modification data (data(histone_h3k4me1))
- gene ID conversion inputs (data(genes_for_conversiontest))

## Outputs

- scored PvalueAnnotation object with per-gene combined p-value scores
- SMITE network modules (spin-glass modules on REACTOME, maxsize=50)
- module KEGG/pathway annotations via goseq-like approach
- module comparison plots (BioNet/plotModule-style rendering, e.g. fem_smite_comparison.png)
- KS-test enrichment statistics comparing SMITE-R, SMITE-F, and FEM module gene-score distributions

## How to apply

Load methylation data and curated RNA-seq expression data from the SMITEdatasets package, then remove NAs from the p-value column and replace zeros with the minimum non-zero p-value before annotation. Annotate expression and methylation onto a gene-level PvalueAnnotation via annotateExpression and annotateModification (weight_by_method='Stouffer', methylation_promoter='decrease'), normalize p-values with normalizePval(method='rescale', ref='expression'), and combine with scorePval using weights such as methylation_promoter=0.3. Detect modules with igraph's spinglass algorithm on the REACTOME interactome (maxsize=50, num_iterations=1000), then annotate modules with a goseq-like KEGG approach and visualize with BioNet. To benchmark against FEM, run FEM (DoFEM.bi) on the same matched data using CpG methylation averaged within 200 bp of the TSS, first exon, then 1500 bp around the TSS, detect modules on the REACTOME interactome, compute gene-level overlaps, and compare module gene-score distributions with two-sided Kolmogorov-Smirnov tests. Rationale: SMITE accounts for correlation between non-independent values, allowing individually under-powered data sets to be integrated and modules to implicate network members lacking individually significant evidence.

## Related tools

- **SMITE** (R/Bioconductor package that annotates expression and methylation p-values, combines them with correlation correction, and detects network modules via spin-glass) — https://github.com/GreallyLab/SMITE
- **SMITEdatasets** (provides the curated methylation, expression, hg19 annotation, histone, and REACTOME igraph data objects used as inputs) — https://github.com/GreallyLab/SMITEdatasets
- **igraph** (supplies the spinglass algorithm used to determine the best modules on the REACTOME interactome)
- **FEM (Functional Epigenetic Modules)** (benchmark spin-glass method integrating gene expression and epigenetic data for module comparison)
- **BioNet** (visualizes detected modules using interactome data)
- **goseq** (goseq-like approach to annotate modules with pathways such as KEGG)

## Examples

```
library(SMITE); data(methylation); data(curated_expressiondata); test_annotation <- annotateModification(test_annotation, methylation, weight_by_method="Stouffer", weight_by_data=list(promoter=1)); test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3, methylation_body=.2, expression=.5))
```

## Evaluation signals

- Preprocessing invariant: methylation rows with NA in the p-value column (column 5) are removed and zero p-values are replaced by the minimum non-zero p-value before scoring.
- Module recovery check: FEM on the T. gondii HFF data yields 7 modules covering 175 genes, matching the published result.
- Overlap counts: gene-level overlap between FEM and SMITE modules equals 8 genes for SMITE-R and 23 genes for SMITE-F.
- KS-test reproduction: two-sided KS p-values match the reported values — SMITE-R and SMITE-F p = 0.00001, FEM p = 0.2153, and p-value-count bias assessment FEM KS p < 10^-12.
- Figure check: rendering the Fig. 8-style side-by-side module comparison with compare_plot = TRUE produces fem_smite_comparison.png.

## Limitations

- FEM modules were not enriched for high-scoring genes (KS p = 0.2153), so FEM-style spin-glass integration may miss dysregulated subnetworks that SMITE detects.
- Integrative pipelines introduce bias when data sets are pruned — e.g. focusing only on loci with the most significant effects or requiring pairwise comparisons with progressively smaller intersections.
- SMITE modules can implicate network members with no statistically significant individual evidence, so module membership alone is not per-gene significance.

## Evidence

- [abstract] SMITE combines p-values accounting for correlation between non-independent values, assigning significance to genes and gene modules in an interaction network: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values"
- [abstract] FEM benchmark framing: "SMITE outperforms Functional Epigenetic Modules (FEM), the current paradigm of using the spin-glass algorithm to integrate gene expression and epigenetic data"
- [methods] methylation p-value preprocessing: "## Remove NAs from p-value column ##
methylation <- methylation[-which(is.na(methylation[, 5])), ]"
- [methods] module detection algorithm: "we use igraph's spinglass algorithm to determine the best modules"
- [abstract] weighting rationale: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [discussion] modules implicate members without individual significance: "Modules implicate potentially important network members for which there may be no statistically significant evidence."
