---
name: gene-identifier-mapping-for-rem-retrieval
description: Use when you have a gene list (e.g. DE genes from Seurat FindAllMarkers)
  and need the REMs associated to those genes, with cell type/tissue-specific filtering
  via EpiRegio.
when_to_use_negative:
- You already have REM regions (e.g. ChIP-seq peaks as a BED file) and want the associated
  genes — use EpiRegio's Region Query with a defined overlap percentage instead of
  the Gene Query.
- Your input is a set of DNA sequences or regions rather than gene identifiers — the
  Gene Query starts from genes, not regions ('In contrast to existing approaches,
  it starts with a gene and looks for REMs, not viceversa').
edam_operation: http://edamontology.org/operation_2422
tools:
- name: EpiRegio
  role: Gene Query (web form or REST API) retrieves the REMs associated to the submitted
    gene list, with optional 'Filter for cell types/tissues' and Cell type score thresholds;
    results are downloaded as CSV.
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: Seurat
  role: 'Produces the input gene list: FindAllMarkers with Wilcoxon Rank Sum test
    on the TGF-beta-treated versus untreated HUVEC single-cell RNA-seq data, retaining
    genes with p <= 0.01.'
- name: bedtools
  role: Extracts the DNA sequences of the identified REMs (bedtools getfasta) for
    downstream motif enrichment with PASTAA.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:acec5c56c9acfbc61f8441d6881e29da58fa9097f8d81dfbb26c272d51df4c76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - Seurat
  - bedtools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-identifier-mapping-for-rem-retrieval

## Summary

Maps a list of genes of interest (e.g. differentially expressed genes from a single-cell RNA-seq comparison) to their associated regulatory elements (REMs) via EpiRegio's Gene Query, optionally filtered by cell type/tissue, and exports the REM table for downstream validation and analysis.

## When to use

Use when you have a gene list (e.g. DE genes from Seurat FindAllMarkers) and need the REMs associated to those genes, with cell type/tissue-specific filtering via EpiRegio.

## When NOT to use

- You already have REM regions (e.g. ChIP-seq peaks as a BED file) and want the associated genes — use EpiRegio's Region Query with a defined overlap percentage instead of the Gene Query.
- Your input is a set of DNA sequences or regions rather than gene identifiers — the Gene Query starts from genes, not regions ('In contrast to existing approaches, it starts with a gene and looks for REMs, not viceversa').

## Inputs

- Gene list file GeneSet.txt (one gene identifier per line, formatted like the example file in ApplicationScenarioExamples/identifyEnrichedTFs)
- Cell type/tissue filter term (e.g. 'heart') for EpiRegio Gene Query

## Outputs

- CSV table of REMs associated to the input genes (e.g. 11,836 REMs for 304 DE genes under cond_tgfb_huvec_endomt), including Cell type score and Cell type DNase1 signal columns
- Optionally, REM DNA sequences extracted with bedtools getfasta for downstream motif enrichment

## How to apply

Export the significant differentially expressed gene list (e.g. genes with Wilcoxon Rank Sum P <= 0.01 from Seurat's FindAllMarkers) as GeneSet.txt matching the format of the example file in the ApplicationScenarioExamples/identifyEnrichedTFs repository folder. Submit the gene list to EpiRegio's Gene Query (web form at https://epiregio.de/geneQuery/ or REST API at https://epiregio.de/REST_API/GeneQuery/<input>/) to identify the REMs associated to the genes of interest. Because endothelial cells within the heart undergo EndoMT and the regulatory processes are expected to be comparable, enter 'heart' in the 'Filter for cell types/tissues' field. Export the resulting REM table by clicking the CSV button, then validate that the output contains the expected number of REMs per number of input genes — a substantial shortfall indicates incorrect gene identifiers or filtering settings. Optionally continue with DNA-sequence extraction of the REMs using bedtools for motif enrichment steps such as PASTAA.

## Related tools

- **EpiRegio** (Gene Query (web form or REST API) retrieves the REMs associated to the submitted gene list, with optional 'Filter for cell types/tissues' and Cell type score thresholds; results are downloaded as CSV.) — https://github.com/TeamRegio/EpiRegioDB
- **Seurat** (Produces the input gene list: FindAllMarkers with Wilcoxon Rank Sum test on the TGF-beta-treated versus untreated HUVEC single-cell RNA-seq data, retaining genes with p <= 0.01.)
- **bedtools** (Extracts the DNA sequences of the identified REMs (bedtools getfasta) for downstream motif enrichment with PASTAA.)

## Examples

```
curl -s "https://epiregio.de/REST_API/GeneQuery/$(paste -sd, GeneSet.txt)/" > rem_table.csv
```

## Evaluation signals

- The exported Gene Query CSV contains the reported 11,836 REMs for the 304 differentially expressed genes (metric_de_genes and metric_rems_de_genes under cond_tgfb_huvec_endomt).
- A substantial shortfall in returned REMs relative to the reference count indicates incorrect gene identifiers or filtering settings.
- The input GeneSet.txt matches the format of the example file in the ApplicationScenarioExamples/identifyEnrichedTFs repository folder.
- Every returned REM row carries the cell type/tissue filter (e.g. 'heart') applied via the 'Filter for cell types/tissues' field.
- The resulting REM table includes sortable Cell type score values consistent with the documented REM ranking scheme.

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs.
- Identifying REMs is difficult, as there is no method yet to locate them with absolute certainty; REM-gene associations are inferred from chromatin accessibility and gene expression variation.
- Using a surrogate tissue filter (e.g. 'heart' for endothelial cells) relies on the assumption that the regulatory processes are comparable across cell types/tissues.
- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, etc.), which complicates exact reproduction.

## Evidence

- [methods] Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest: "Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest"
- [methods] heart filter step: "Enter *heart* in the field *Filter for cell types/tissues*"
- [methods] CSV export step: "Download the resulting table by clicking on the bottom *CSV*."
- [readme] example gene list location: [evidence span withheld — anchor status: external_doc]
