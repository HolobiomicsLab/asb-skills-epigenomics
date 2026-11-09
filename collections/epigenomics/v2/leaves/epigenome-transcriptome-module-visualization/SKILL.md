---
name: epigenome-transcriptome-module-visualization
description: Use when SMITE modules have already been identified (runSpinglass/runBioNet)
  from scored gene networks and you need to assign pathway meaning and render interpretable
  subnetwork figures for reporting.
when_to_use_negative:
- Input is a raw scored PvalueAnnotation with no modules yet — run runSpinglass or
  runBioNet first, since there is nothing to annotate or plot
- The goal is de novo pathway discovery rather than annotating pre-defined network
  modules — a standard goseq/gene-set enrichment run on a ranked gene list is simpler
  and more appropriate
- The interactome lacks symbol-level gene identifiers matching the module genes —
  mapping errors will produce empty or misleading module plots
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: SMITE
  role: Provides runGOseq, searchGOseq, and plotModule to annotate modules with KEGG
    pathways, search them by keyword, and draw the module networks
  repo: https://github.com/GreallyLab/SMITE
- name: goseq
  role: Underlying pathway-annotation approach run through SMITE's runGOseq to annotate
    modules using pathways like KEGG
- name: BioNet
  role: Module-building package used with the interactome to visualize modules; its
    Heinz-derived modules can be compared with spin-glass results before annotation/plotting
- name: igraph
  role: Spin-glass algorithm (via runSpinglass) that produced the 13-module partitions
    on the REACTOME interactome which are subsequently annotated and plotted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-017-1477-3
    title: 'SMITE: an R/Bioconductor package that identifies network modules by integrating
      genomic and epigenomic information'
  source_content_hash: sha256:b03177a21f8829b4e2d4ca506ec84a10d1f59b1a6baffdc9db4cd28101a7d245
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# epigenome-transcriptome-module-visualization

## Summary

Annotate and visualize SMITE network modules identified by the spin-glass (igraph) or BioNet/Heinz algorithms from integrated transcriptome/epigenome scores, using KEGG/pathway enrichment (runGOseq) and network plots (plotModule) to interpret dysregulated subnetworks such as cell-cycle/MAPK and chromatin-regulation modules.

## When to use

Use when SMITE modules have already been identified (runSpinglass/runBioNet) from scored gene networks and you need to assign pathway meaning and render interpretable subnetwork figures for reporting.

## When NOT to use

- Input is a raw scored PvalueAnnotation with no modules yet — run runSpinglass or runBioNet first, since there is nothing to annotate or plot
- The goal is de novo pathway discovery rather than annotating pre-defined network modules — a standard goseq/gene-set enrichment run on a ranked gene list is simpler and more appropriate
- The interactome lacks symbol-level gene identifiers matching the module genes — mapping errors will produce empty or misleading module plots

## Inputs

- PvalueAnnotation object with SMITE scores and module assignments from runSpinglass or runBioNet
- Module gene lists from SMITE-R and SMITE-F weighting models (e.g. 528 and 510 genes across 13 modules each)
- Coverage table such as hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz loaded via read.table from the SMITE extdata directory
- REACTOME interactome igraph (Reactome.Symbol.Igraph.rda from SMITE package data)

## Outputs

- KEGG/pathway enrichment annotations per module from runGOseq
- Keyword-matched module annotations from searchGOseq (e.g. 'cell cycle')
- Rendered module network plots (PNG files) from plotModule, e.g. cell-cycle/MAPK and chromatin-regulation modules corresponding to Figs. 6-7
- Summary table (e.g. smite_modules.tsv) with module assignments, sizes, overlaps, and KEGG annotations

## How to apply

Take the scored PvalueAnnotation objects (e.g. SMITE-R with expression + methylation_promoter, or SMITE-F adding methylation_body and methylation_h3k4me1) and their module assignments from runSpinglass (spin-glass parameter 0.5, module size 8-100 genes) or runBioNet on the REACTOME interactome. Annotate each module with runGOseq using type='kegg' and a coverage set such as the hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz table to correct for assay coverage. Use searchGOseq with a keyword string like 'cell cycle' to locate relevant modules among the 13 identified per model, then draw the subnetwork with plotModule, coloring genes by their SMITE scores and effect directions. Write module assignments, sizes, overlaps, and KEGG annotations (e.g. to smite_modules.tsv) and save module plots as PNG files for comparison across weighting models (e.g. the 94-gene overlap between SMITE-R's 528 and SMITE-F's 510 module genes).

## Related tools

- **SMITE** (Provides runGOseq, searchGOseq, and plotModule to annotate modules with KEGG pathways, search them by keyword, and draw the module networks) — https://github.com/GreallyLab/SMITE
- **goseq** (Underlying pathway-annotation approach run through SMITE's runGOseq to annotate modules using pathways like KEGG)
- **BioNet** (Module-building package used with the interactome to visualize modules; its Heinz-derived modules can be compared with spin-glass results before annotation/plotting)
- **igraph** (Spin-glass algorithm (via runSpinglass) that produced the 13-module partitions on the REACTOME interactome which are subsequently annotated and plotted)

## Examples

```
coverage <- read.table(system.file("extdata", "hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz", package="SMITE")); test_annotation <- runGOseq(pvalue_annotation=test_annotation, coverage=coverage, type="kegg"); searchGOseq(test_annotation, string="cell cycle"); plotModule(test_annotation, module_number=3)
```

## Evaluation signals

- Each of the 13 modules per weighting model receives non-empty KEGG annotations from runGOseq and the output table lists module sizes matching the 528-gene (SMITE-R) and 510-gene (SMITE-F) totals
- searchGOseq with the string 'cell cycle' returns the expected cell-cycle/MAPK module corresponding to Figs. 6-7
- Overlap computed between SMITE-R and SMITE-F module gene sets equals 94 genes
- plotModule renders network figures for the identified cell-cycle/MAPK and chromatin-regulation modules, saved as PNG files
- KEGG annotation coverage is adjusted using the hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz coverage table, so enrichment p-values reflect assay-covered genes rather than all genes

## Limitations

- Module composition is highly weighting-model dependent: SMITE-R and SMITE-F each found 13 modules but only 94 genes overlapped between the two models' modules, so annotations and plots are model-specific
- KEGG-based annotation via goseq requires an appropriate coverage set; uncorrected enrichment can bias toward well-covered genes
- SMITE modules can implicate network members with no statistically significant individual evidence, so per-gene significance should not be read directly from module plots

## Evidence

- [methods] We can then run a `goseq` like approach on our modules to annotate them using pathways, like `KEGG`: "We can then run a `goseq` like approach on our modules to annotate them using pathways, like `KEGG`"
- [methods] we can visualize modules using interactome data and module-building packages, such as `BioNet`: "we can visualize modules using interactome data and module-building packages, such as `BioNet`"
- [methods] we use igraph's spinglass algorithm to determine the best modules: "we use igraph's spinglass algorithm to determine the best modules"
- [discussion] Modules implicate potentially important network members for which there may be no statistically significant evidence.: "Modules implicate potentially important network members for which there may be no statistically significant evidence."
- [methods] coverage=read.table(system.file(
    "extdata", "hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz", package="SMITE")): "coverage=read.table(system.file(
    "extdata", "hg19_symbol_hpaii.sites.inbodyand2kbupstream.bed.gz", package="SMITE"))"
