---
name: differential-gene-expression-analysis
description: Use when cluster assignments from an unsupervised expression clustering
  solution (e.g., SRIQ, 6 clusters at diameter 0.63) exist and per-cluster up-/down-regulated
  gene lists and GO terms are needed.
when_to_use_negative:
- Expression data are not log2-transformed (and off-set by 0.1 or with values <1 set
  to 1), as required before SAM analysis
- No cluster solution exists yet — run SRIQ clustering and EXTRACT clusters first
- Goal is assigning samples into clusters rather than characterizing existing SRIQ
  clusters by DEG/enrichment
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: SRIQ
  role: Pipeline providing SAMDEG (SAM-based one-versus-rest DEG analysis of core
    clusters) and the EnrichR gene enrichment module
  repo: https://github.com/StaafLab/SRIQ
- name: Java
  role: Runtime for SRIQS.jar / SAMDEG.jar used to execute SAMDEG (e.g., with -Xmx8g)
- name: Python
  role: Jupyter analysis_pipeline handling gene symbol mapping via the mygene API
    and EnrichR enrichment of DEG lists
  repo: https://github.com/StaafLab/SRIQ
- name: EnrichR
  role: GO database enrichment analysis on per-cluster up- and down-regulated gene
    lists
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.03.036
    title: 'SRIQ clustering: A fusion of Random Forest, QT clustering, and KNN concepts'
  source_content_hash: sha256:566cdedf9d211fc9b6d53a6a9166478e78022b0d898a97ab5759c8db7a5de2fd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/StaafLab/SRIQ
  repo_url: https://github.com/StaafLab/SRIQ
  tools:
  - EnrichR
  - Java
  - Python
  - SRIQ
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs one-versus-rest differential gene expression per SRIQ core cluster using SAM permutation testing (SAMDEG), then maps genes to symbols and runs EnrichR GO enrichment to characterize transcriptional programs of tumor subgroups in TCGA LUAD.

## When to use

Use when cluster assignments from an unsupervised expression clustering solution (e.g., SRIQ, 6 clusters at diameter 0.63) exist and per-cluster up-/down-regulated gene lists and GO terms are needed.

## When NOT to use

- Expression data are not log2-transformed (and off-set by 0.1 or with values <1 set to 1), as required before SAM analysis
- No cluster solution exists yet — run SRIQ clustering and EXTRACT clusters first
- Goal is assigning samples into clusters rather than characterizing existing SRIQ clusters by DEG/enrichment

## Inputs

- Log2-transformed, tab-separated gene expression matrix (e.g., newFiltered_35k.txt; Gene x Sample)
- Cluster assignments extracted for the chosen SRIQ solution (diameter 0.63, 6 clusters)
- test.properties file configured with studyName, studyPath, inFileName, outPath
- SAMDEG parameters: spiral flag, diameter, number of clusters, q-value (0), fold-change (2)

## Outputs

- Per-cluster differentially expressed gene lists with SAM q-value and fold-change
- EnrichR GO enrichment terms with statistics for up- and down-regulated gene sets
- Compiled single table of per-cluster DEGs and enrichment terms (e.g., under Results_log_0.63_6/)

## How to apply

Load the log2-transformed expression matrix (e.g., newFiltered_35k.txt, genes as rows, samples as columns) together with the extracted cluster assignments for the chosen SRIQ solution (e.g., diameter 0.63, 6 clusters). For each cluster, run SAMDEG in one-versus-rest mode with q-value 0 and fold-change 2, which applies significance analysis of microarrays (SAM) with permutation testing to call up- and down-regulated genes. Because the EnrichR module assumes gene symbols, map input identifiers (e.g., ENSG IDs) to symbols via the mygene API, setting the 'scopes' variable to the input identifier format. Run EnrichR against a GO database on each cluster's DEG list and compile DEG rows (with SAM q-value and fold-change fields) plus enrichment statistics into a single table for all clusters.

## Related tools

- **SRIQ** (Pipeline providing SAMDEG (SAM-based one-versus-rest DEG analysis of core clusters) and the EnrichR gene enrichment module) — https://github.com/StaafLab/SRIQ
- **Java** (Runtime for SRIQS.jar / SAMDEG.jar used to execute SAMDEG (e.g., with -Xmx8g))
- **Python** (Jupyter analysis_pipeline handling gene symbol mapping via the mygene API and EnrichR enrichment of DEG lists) — https://github.com/StaafLab/SRIQ
- **EnrichR** (GO database enrichment analysis on per-cluster up- and down-regulated gene lists)

## Examples

```
java -jar -Xmx8g SRIQS.jar SAMDEG test.properties false 0.63 6 0 2 newFiltered_35k.txt
```

## Evaluation signals

- Output table contains DEG rows for each of the 6 clusters with SAM q-value and fold-change fields populated
- Enrichment terms carry EnrichR statistics for both up- and down-regulated gene sets
- Cluster 4 shows up-regulated DNA replication, mitotic spindle and cell cycle GO terms; cluster 1 shows enriched immune response terms
- All gene identifiers passed to EnrichR are gene symbols (mygene mapping from scopes=ENSG completed without unmapped entries)
- Output written under the expected path, e.g., ...\QC_Spiral(false)\Results_log_0.63_6\

## Limitations

- Cluster solutions are sensitive to the diameter parameter, which must be evaluated against stability and occurrence across iterations before DEG analysis
- SRIQ cannot detect small or underrepresented biological subgroups in smaller datasets, so small clusters may yield unreliable DEGs
- SRIQ computation and memory intensive with large bag size, many permutations, and large datasets (samples and features)
- SAMDEG requires log2-transformed input (off-set by 0.1 or values <1 set to 1), distinct from the additional median-centering required for clustering

## Evidence

- [results] One-versus-rest SAM on core clusters: [evidence span withheld — anchor status: not_found]
- [readme] mygene symbol mapping for EnrichR: [evidence span withheld — anchor status: not_found]
