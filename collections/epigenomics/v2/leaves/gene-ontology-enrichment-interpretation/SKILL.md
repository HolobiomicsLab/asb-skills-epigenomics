---
name: gene-ontology-enrichment-interpretation
description: Use when you have a ranked list of genes from EpiAlign differential alignment
  (Wilcoxon P-values comparing within-tissue vs cross-tissue alignment scores) and
  need functional interpretation via GO terms or validation against known tissue-specific
  gene labels.
when_to_use_negative:
- The input is raw multi-track epigenomic signal matrices or uncompressed chromatin
  state sequences — run EpiAlign's compressed Smith-Waterman alignment and Wilcoxon
  ranking first; GO enrichment only consumes ranked gene lists.
- The goal is to compare chromatin state sequences or epigenomic samples directly
  rather than functionally interpret an already-ranked gene list.
- The gene list is not ranked by a statistical criterion (e.g. differential alignment
  P-values) — GOrilla expects genes ordered by score for its enrichment model.
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: EpiAlign
  role: Produces the ranked differential alignment P-values per protein-coding gene
    that feed the GO enrichment and hypergeometric validation
  repo: https://github.com/zzz3639/EpiAlign
- name: GOrilla
  role: Performs the GO enrichment analysis on the top-ranked aligned genes, extracting
    terms with P < 0.0001
- name: ChromHMM
  role: Upstream chromatin state annotation that encodes multi-track epigenomic signals
    into the single-track state sequences EpiAlign aligns
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:225e5a5d33d6cf723410929c1f7a45f1638a0b6a7591240db9eac1c36b9ea0f5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  - GOrilla
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-ontology-enrichment-interpretation

## Summary

Interpret ranked gene lists produced by EpiAlign differential chromatin-state alignment by running Gene Ontology (GO) enrichment (e.g. GOrilla) on the top genes with the smallest P-values, and validating enrichment against known tissue-specific gene sets. This connects alignment-derived chromatin patterns to biological function.

## When to use

Use when you have a ranked list of genes from EpiAlign differential alignment (Wilcoxon P-values comparing within-tissue vs cross-tissue alignment scores) and need functional interpretation via GO terms or validation against known tissue-specific gene labels.

## When NOT to use

- The input is raw multi-track epigenomic signal matrices or uncompressed chromatin state sequences — run EpiAlign's compressed Smith-Waterman alignment and Wilcoxon ranking first; GO enrichment only consumes ranked gene lists.
- The goal is to compare chromatin state sequences or epigenomic samples directly rather than functionally interpret an already-ranked gene list.
- The gene list is not ranked by a statistical criterion (e.g. differential alignment P-values) — GOrilla expects genes ordered by score for its enrichment model.

## Inputs

- Ranked gene list with differential alignment P-values from EpiAlign (two-sample one-sided Wilcoxon rank-sum test between within-tissue and cross-tissue alignment score groups)
- Protein-coding gene coordinates from GENCODE v10 (Roadmap-compatible, chromosome Y genes excluded)
- Brain/heart-specific gene labels for hypergeometric validation

## Outputs

- GO enrichment terms with P < 0.0001 for the top-ranked aligned genes
- Hypergeometric enrichment counts and P-values for tissue-specific genes in the top 100 (expected ~9 brain-specific and ~4 heart-specific, P < 10^-30)
- Ranked gene table with STMN4 as the top (smallest P-value) hit

## How to apply

After EpiAlign ranks all protein-coding genes (e.g. the 19,935 GENCODE v10 protein-coding genes compatible with the Roadmap database, excluding chromosome Y) by two-sample one-sided Wilcoxon rank-sum differential alignment P-values, take the top-ranked genes and perform GO enrichment analysis on the genes receiving the smallest P-values. In the brain-vs-heart Roadmap analysis (E071 brain vs heart sample), GOrilla is used to extract GO terms with P < 0.0001 from the top-ranked aligned genes. Independently validate biological relevance by counting brain-specific and heart-specific genes among the top 100 ranked genes and testing hypergeometric enrichment against known tissue-specific gene labels — the reported result is ~9 brain-specific and ~4 heart-specific genes with hypergeometric P < 10^-30, with STMN4 receiving the top (smallest) P-value.

## Related tools

- **EpiAlign** (Produces the ranked differential alignment P-values per protein-coding gene that feed the GO enrichment and hypergeometric validation) — https://github.com/zzz3639/EpiAlign
- **GOrilla** (Performs the GO enrichment analysis on the top-ranked aligned genes, extracting terms with P < 0.0001)
- **ChromHMM** (Upstream chromatin state annotation that encodes multi-track epigenomic signals into the single-track state sequences EpiAlign aligns)

## Evaluation signals

- STMN4 receives the top (smallest) differential alignment P-value in the brain-vs-heart ranking
- Hypergeometric test on the top 100 genes yields ~9 brain-specific and ~4 heart-specific genes with P < 10^-30
- GOrilla returns GO terms at P < 0.0001 for the top-ranked aligned genes
- Ranked input was restricted to Roadmap-compatible protein-coding genes from GENCODE v10, excluding chromosome Y
- Differential P-values were computed with a two-sample one-sided Wilcoxon rank-sum test comparing within-brain vs brain-versus-heart alignment score groups per gene

## Limitations

- Enrichment counts (~9 brain-specific, ~4 heart-specific in the top 100) are tied to the specific Roadmap E071 brain vs heart comparison and may not transfer to other tissue pairs
- GO enrichment thresholds (P < 0.0001) and the top-200-gene input window reflect this paper's workflow; other windows may change resulting terms
- The upstream EpiAlign repository README notes that downloading the package as a zip file may cause errors; git clone is recommended instead, which affects reproducibility of the ranked gene list feeding enrichment

## Evidence

- [results] We next perform the Gene Ontology (GO) enrichment analysis (30) on the top 200 genes that receive the smallest P-values: "We next perform the Gene Ontology (GO) enrichment analysis (30) on the top 200 genes that receive the smallest P-values"
- [other] Perform GOrilla GO enrichment on the top-ranked aligned genes and extract terms with P < 0.0001: [evidence span withheld — anchor status: not_found]
- [results] we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [readme] README advises git clone because zip download may cause errors: "if an error occurs,  we recommend you to download our package by "git clone" ruther than downloading the zip file."
