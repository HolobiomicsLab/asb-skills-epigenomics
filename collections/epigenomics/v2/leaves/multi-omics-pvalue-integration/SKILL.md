---
name: multi-omics-pvalue-integration
description: Use when combining gene expression p-values with epigenomic modification
  p-values (promoter methylation, gene-body methylation/hydroxymethylation, enhancer
  H3K4me1) that are anchored to hg19 TSS coordinates, to obtain per-gene scores and
  significant subnetwork modules.
when_to_use_negative:
- Only a single omics data type is available (e.g. expression alone) with no methylation/modification
  p-value track to combine — a plain differential-expression workflow suffices
- Your data are not loci anchored to gene annotations (TSS-based promoter/enhancer
  mapping is required by makePvalueAnnotation)
- You already have module-level subnetworks and only need visualization; the scoring
  pipeline would be redundant
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: SMITE
  role: Core R/Bioconductor package implementing makePvalueAnnotation, annotateExpression,
    annotateModification, makePvalueObject, normalizePval, scorePval and module scoring
  repo: https://github.com/GreallyLab/SMITE
- name: igraph
  role: Provides the spin-glass algorithm used by runSpinglass to determine the best
    modules on the REACTOME interactome
- name: BioNet
  role: Alternative module-building package (Heinz algorithm via runBioNet) used to
    visualize and compare modules with the spin-glass results
- name: goseq
  role: goseq-like pathway over-representation used by runGOseq to annotate modules
    with KEGG pathways
- name: R
  role: Runtime environment for the SMITE workflow (library(SMITE))
provenance:
  source_task_ids:
  - task_006
  - task_008
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:bedd7099dba0e061868b2242eddede092afd62cc5f42806c0dde669f260ffabb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - BioNet
  - R
  - SMITE
  - goseq
  - igraph
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-omics-pvalue-integration

## Summary

Integrates transcriptional and epigenomic p-values (gene expression, promoter/body/h3k4me1 methylation) into a single per-gene score with SMITE by correlation-aware Stouffer weighting followed by Fisher's method, then assigns significance to network modules on an interactome. It allows weighting each data type so individually under-powered datasets can still contribute to module discovery.

## When to use

Use when combining gene expression p-values with epigenomic modification p-values (promoter methylation, gene-body methylation/hydroxymethylation, enhancer H3K4me1) that are anchored to hg19 TSS coordinates, to obtain per-gene scores and significant subnetwork modules.

## When NOT to use

- Only a single omics data type is available (e.g. expression alone) with no methylation/modification p-value track to combine — a plain differential-expression workflow suffices
- Your data are not loci anchored to gene annotations (TSS-based promoter/enhancer mapping is required by makePvalueAnnotation)
- You already have module-level subnetworks and only need visualization; the scoring pipeline would be redundant

## Inputs

- Curated gene expression table (gene ID, effect size in column 1, p-value in column 2)
- Methylation/modification table with NA rows removed and zeros replaced by the minimum non-zero p-value
- hg19 gene annotation BED (hg19_genes_bed)
- H3K4me1 ChIP-seq BED as other_tss_distance enhancer data (histone_h3k4me1)
- REACTOME symbol-level igraph interactome (Reactome.Symbol.Igraph.rda)
- CpG coverage BED (hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz) for goseq coverage

## Outputs

- PvalueAnnotation object with component scores (expression, methylation_promoter, methylation_body, methylation_h3k4me1) and a final per-gene score
- Table of per-gene combined p-values and G = -2*ln(p) statistics (e.g. scored_gene_pvalues.tsv)
- Network modules with gene assignments, sizes and overlaps (e.g. smite_modules.tsv)
- KEGG-annotated modules and module plots (PNG) via runGOseq/searchGOseq/plotModule

## How to apply

Build a PvalueAnnotation with makePvalueAnnotation using promoter_upstream_distance=1000, promoter_downstream_distance=1000 and other_tss_distance=5000, attach expression via annotateExpression (effect_col=1, pval_col=2) and methylation via annotateModification with weight_by_method="Stouffer", weight_by="distance" for promoter, body and h3k4me1 contexts. Set a priori effect directions with makePvalueObject (methylation_promoter="decrease", methylation_body="increase", methylation_h3k4me1="bidirectional") and normalize p-values with normalizePval(ref="expression", method="rescale"). Combine component scores with scorePval using Stouffer's method with weights (e.g. expression=0.3, methylation_promoter=0.3, methylation_body=0.1, methylation_h3k4me1=0.3), estimating per-gene p-values by Monte Carlo (MCM) bootstrapping to account for correlation between non-independent values. Convert each per-gene p-value to a final score G_i = -2*ln(p_i), which is approximately Chi-square distributed with 2 degrees of freedom (2k df for a module of k genes). Assign module significance on the REACTOME interactome by running igraph's spin-glass algorithm (parameter 0.5, modules restricted to 8-100 genes) or BioNet/Heinz, then annotate modules with a goseq-like approach using KEGG pathways. Rationale: multiple event types at one locus increase confidence in regulatory activity, and weighting permits integration of individually under-powered datasets without pruning loci.

## Related tools

- **SMITE** (Core R/Bioconductor package implementing makePvalueAnnotation, annotateExpression, annotateModification, makePvalueObject, normalizePval, scorePval and module scoring) — https://github.com/GreallyLab/SMITE
- **igraph** (Provides the spin-glass algorithm used by runSpinglass to determine the best modules on the REACTOME interactome)
- **BioNet** (Alternative module-building package (Heinz algorithm via runBioNet) used to visualize and compare modules with the spin-glass results)
- **goseq** (goseq-like pathway over-representation used by runGOseq to annotate modules with KEGG pathways)
- **R** (Runtime environment for the SMITE workflow (library(SMITE)))

## Examples

```
test_annotation <- scorePval(test_annotation, weights=c(methylation_promoter=.3, methylation_body=.1, expression=.3, methylation_h3k4me1=.3))
```

## Evaluation signals

- Every output per-gene p-value lies in (0,1]
- The distribution of G = -2*ln(p) statistics matches a Chi-square distribution with 2 degrees of freedom by visual/quantile check
- Module-level Fisher p-values are computed as Chi-square with 2k degrees of freedom for k genes per module
- Weights in scorePval match the specified model (e.g. 0.3 expression / 0.3 promoter / 0.1 body / 0.3 h3k4me1) and the score directionality flags match makePvalueObject
- On the T. gondii HFF benchmark, spin-glass modules restricted to 8-100 genes yield 13 modules covering 528 (reduced) and 510 (full) genes with a 94-gene overlap

## Limitations

- Effect directions (e.g. promoter methylation 'decrease', body 'increase', h3k4me1 'bidirectional') must be specified a priori; relationships opposite to these assumptions are penalized
- P-values must be cleaned first: remove NA rows and replace zeros with the minimum non-zero p-value, and take the lowest p-value for genes with multiple entries
- Enhancer/modification data must be mappable to genes via TSS distances (promoter ±1000 bp, other_tss_distance 5000 bp), which constrains suitable inputs
- Per-gene p-values come from Monte Carlo sampling, so they are estimates rather than exact statistics

## Evidence

- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets"
- [abstract] The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets: "The contribution of each type of genomic data can be weighted, permitting integration of individually under-powered data sets"
- [methods] we use igraph's spinglass algorithm to determine the best modules: "we use igraph's spinglass algorithm to determine the best modules"
- [methods] We can then run a `goseq` like approach on our modules to annotate them using pathways, like `KEGG`: "We can then run a `goseq` like approach on our modules to annotate them using pathways, like `KEGG`"
- [discussion] Thus, SMITE is a discovery platform to integrate multi-level genomic observations that represents a significant improvement over existing integrative genomics approaches.: "Thus, SMITE is a discovery platform to integrate multi-level genomic observations that represents a significant improvement"
