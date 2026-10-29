---
name: cell-type-specific-rem-filtering
description: Use when submitting a gene list to EpiRegio's Gene Query and you need
  REMs restricted to a relevant cell type or tissue context rather than genome-wide
  REMs.
when_to_use_negative:
- Running a Region Query with TF ChIP-seq peaks (e.g. ENCODE BED files), where the
  relevant filter is the 'Overlap percentage' setting, not a cell-type/tissue filter
- You need all REMs linked to the genes regardless of cellular context, e.g. for genome-wide
  annotation rather than cell-type-specific interpretation
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3173
tools:
- name: EpiRegio
  role: Web server / REST API whose Gene Query retrieves REMs associated to input
    genes, filtered via the 'Filter for cell types/tissues' field
  repo: https://epiregio.de/
- name: EpiRegioDB
  role: Repository and documentation backing the EpiRegio database and its use cases
  repo: https://github.com/TeamRegio/EpiRegioDB
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:68f0b433ae9fdac8382cbf29cafa8b915633cb6f554d4a1d9df67f324ae605ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiRegio
  - EpiRegioDB
  - Seurat
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cell-type-specific-rem-filtering

## Summary

Narrow EpiRegio Gene Query results to REMs active in a specific cell type or tissue (e.g. 'heart') by using the 'Filter for cell types/tissues' field, so retrieved REMs are biologically relevant to the cell context of the input gene set.

## When to use

Use when submitting a gene list to EpiRegio's Gene Query and you need REMs restricted to a relevant cell type or tissue context rather than genome-wide REMs.

## When NOT to use

- Running a Region Query with TF ChIP-seq peaks (e.g. ENCODE BED files), where the relevant filter is the 'Overlap percentage' setting, not a cell-type/tissue filter
- You need all REMs linked to the genes regardless of cellular context, e.g. for genome-wide annotation rather than cell-type-specific interpretation

## Inputs

- Gene list of interest (e.g. GeneSet.txt of differentially expressed genes, matching the example format in ApplicationScenarioExamples/identifyEnrichedTFs)
- Cell type/tissue keyword to filter on (e.g. 'heart')
- Optional Cell type score / Cell type DNase1 signal thresholds

## Outputs

- Table of REMs associated to the input genes, restricted to the selected cell type/tissue
- Exported CSV of the filtered REM table

## How to apply

Submit the gene set of interest via EpiRegio's Gene Query (https://epiregio.de/geneQuery/ or REST API https://epiregio.de/REST_API/GeneQuery/<input>/), then enter the relevant tissue keyword — e.g. 'heart' for endothelial cells undergoing endothelial-to-mesenchymal transition (EndoMT) — in the 'Filter for cell types/tissues' field. The rationale is that regulatory processes in the queried cell type are expected to be comparable to those in the matched tissue (e.g. endothelial cells within the heart). Optionally combine with thresholds on the Cell type score or Cell type DNase1 signal; the filtered result can then be exported by clicking the CSV button. In the TGF-beta-treated HUVEC EndoMT scenario, filtering with 'heart' and verifying that the exported CSV contains 11,836 REMs for the 304 differentially expressed genes confirms correct application.

## Related tools

- **EpiRegio** (Web server / REST API whose Gene Query retrieves REMs associated to input genes, filtered via the 'Filter for cell types/tissues' field) — https://epiregio.de/
- **EpiRegioDB** (Repository and documentation backing the EpiRegio database and its use cases) — https://github.com/TeamRegio/EpiRegioDB

## Examples

```
curl -X POST https://epiregio.de/REST_API/GeneQuery/$(cat GeneSet.txt | paste -sd, -) --data 'filter_celltypes=heart' > rems_heart.csv
```

## Evaluation signals

- Exported CSV for the TGF-beta-treated HUVEC EndoMT case (filtered with 'heart') must contain the reported 11,836 REMs
- The number of differentially expressed input genes must equal metric_de_genes = 304 (Seurat FindAllMarkers, Wilcoxon Rank Sum, P ≤ 0.01)
- A substantial shortfall in REM count indicates incorrect gene identifiers or filtering settings
- Every REM row in the output should carry cell type/tissue annotation matching the filter keyword

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs
- Filtering relies on a keyword match against cell types/tissues, so choosing an unmatched or overly narrow tissue term can drop relevant REMs (shortfall in REM count)
- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, etc.)

## Evidence

- [methods] cell-type filter in Gene Query: "Enter *heart* in the field *Filter for cell types/tissues*"
- [methods] Gene Query purpose: "Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest"
- [methods] threshold options: "The thresholds refer to the *Cell type score* and the *Cell type DNase1 signal* of the REMs in the cell types/tissues."
- [abstract] REST API access: "EpiRegio allows programmatic access through a REST API and is freely available at https://epiregio.de/."
- [discussion] STITCHIT limitation: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
