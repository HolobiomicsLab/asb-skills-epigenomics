---
name: bionet-heinz-subnetwork-search
description: Use when SMITE-scored genes (scorePval output on a PvalueAnnotation)
  need to be converted into ranked subnetwork modules on an interactome, or when comparing
  Heinz-derived modules against spin-glass results.
when_to_use_negative:
- The input is raw expression or methylation data that has not yet been annotated
  and scored through makePvalueAnnotation / normalizePval / scorePval — BioNet requires
  per-gene significance scores, not raw assay data
- The goal is de novo module detection without a reference interactome — the Heinz
  search operates on a supplied network such as REACTOME
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: BioNet
  role: Module-building package used to run the Heinz algorithm on the scored network
    and visualize modules from the interactome
  repo: https://github.com/GreallyLab/SMITE
- name: SMITE
  role: Provides the scored PvalueAnnotation (scorePval), the REACTOME igraph, and
    the runBioNet wrapper that drives the subnetwork search
  repo: https://github.com/GreallyLab/SMITE
- name: igraph
  role: Supplies the spin-glass module detection against which Heinz modules are compared
    on the same scored networks
  repo: https://github.com/GreallyLab/SMITE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:5da236c7b7313d31a3bf9c0f01ca830dbe9fc32bd56bb16f864684fc675ee909
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/GreallyLab/SMITE
  tools:
  - BioNet
  - R
  - SMITE
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bionet-heinz-subnetwork-search

## Summary

Uses the BioNet/Heinz algorithm on SMITE-scored gene p-values to identify significant subnetwork modules within an interaction network (e.g. the packaged REACTOME symbol igraph), as a comparison point to igraph's spin-glass module detection in the SMITE workflow.

## When to use

Use when SMITE-scored genes (scorePval output on a PvalueAnnotation) need to be converted into ranked subnetwork modules on an interactome, or when comparing Heinz-derived modules against spin-glass results.

## When NOT to use

- The input is raw expression or methylation data that has not yet been annotated and scored through makePvalueAnnotation / normalizePval / scorePval — BioNet requires per-gene significance scores, not raw assay data
- The goal is de novo module detection without a reference interactome — the Heinz search operates on a supplied network such as REACTOME

## Inputs

- SMITE PvalueAnnotation with scored genes (output of scorePval under SMITE-R or SMITE-F weights)
- REACTOME interactome as a symbol-keyed igraph object (Reactome.Symbol.Igraph.rda from the SMITE package)

## Outputs

- Heinz-derived subnetwork modules (gene sets) per weighting model
- Comparison of BioNet modules with igraph spin-glass modules (e.g. 94-gene overlap between SMITE-R and SMITE-F)
- Module assignments, sizes, overlaps, and KEGG annotations written to smite_modules.tsv

## How to apply

After building a PvalueAnnotation with makePvalueAnnotation, attaching expression (annotateExpression) and methylation (annotateModification with weight_by_method="Stouffer"), and scoring genes with scorePval (e.g. weights 0.3 expression / 0.3 methylation_promoter / 0.1 body / 0.3 enhancer), run the BioNet/Heinz algorithm via runBioNet on the REACTOME interactome (Reactome.Symbol.Igraph.rda from the SMITE package) for each weighting model. In the T. gondii HFF analysis, Heinz modules were compared with igraph spin-glass results (spin-glass parameter 0.5, module size restricted to 8–100 genes) computed on the same scored annotations. The rationale is that SMITE assigns gene significance by combining p-values with correlation correction, and network module-building packages such as BioNet can then visualize and extract dysregulated modules whose significance derives from the cumulative evidence of all members, including genes with no individually significant evidence.

## Related tools

- **BioNet** (Module-building package used to run the Heinz algorithm on the scored network and visualize modules from the interactome) — https://github.com/GreallyLab/SMITE
- **SMITE** (Provides the scored PvalueAnnotation (scorePval), the REACTOME igraph, and the runBioNet wrapper that drives the subnetwork search) — https://github.com/GreallyLab/SMITE
- **igraph** (Supplies the spin-glass module detection against which Heinz modules are compared on the same scored networks) — https://github.com/GreallyLab/SMITE

## Examples

```
test_annotation <- runBioNet(pvalue_annotation = test_annotation, network = Reactome.Symbol.Igraph)
```

## Evaluation signals

- Both SMITE-R and SMITE-F weighting models yield 13 modules each, covering 528 (SMITE-R) and 510 (SMITE-F) genes
- The overlap between SMITE-R and SMITE-F module gene sets equals 94 genes
- Identified modules include the cell-cycle/MAPK and chromatin-regulation modules corresponding to Figs. 6-7, recoverable via searchGOseq with the string 'cell cycle'
- Module assignments, sizes, and KEGG annotations are written to smite_modules.tsv with the expected schema

## Limitations

- Current genomic experiments are underpowered to detect genomic events comprehensively within a network, so module membership depends heavily on the weighting model used for scorePval
- Different weighting models (SMITE-R vs SMITE-F) produce largely non-overlapping modules (only a 94-gene overlap), so results are sensitive to which data types are integrated
- Modules can implicate network members with no statistically significant individual evidence, requiring pathway-level (KEGG/goseq) interpretation rather than per-gene validation

## Evidence

- [methods] we can visualize modules using interactome data and module-building packages, such as `BioNet`: "we can visualize modules using interactome data and module-building packages, such as `BioNet`"
- [abstract] SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules in an interaction network to be assigned significance values: "SMITE combines p-values by accounting for the correlation between non-independent values within data sets, allowing genes and gene modules"
- [discussion] Modules implicate potentially important network members for which there may be no statistically significant evidence.: "Modules implicate potentially important network members for which there may be no statistically significant evidence."
