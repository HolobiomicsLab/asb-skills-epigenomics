---
name: differential-alignment-significance-testing
description: Use when two Roadmap-style chromatin state annotated epigenomes must
  be compared gene-by-gene to identify cell-type-specific genes from chromatin state
  sequence patterns alone (no expression data).
when_to_use_negative:
- The goal is to compare raw multi-track epigenomic signal matrices directly rather
  than reduced single-track chromatin state sequences — EpiAlign requires ChromHMM
  (or similar) state annotation and a compression step first
- The analysis targets horizontal (region-to-region) retrieval of similar 500 kb loci
  within one epigenome instead of gene-level differential alignment between two tissues
- Genes on chromosome Y must be excluded from the comparison per the article's filtering,
  so Y-chromosome-only differential questions are out of scope
edam_operation: http://edamontology.org/operation_0495
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: EpiAlign
  role: Performs the modified Smith-Waterman local alignment on compressed chromatin
    state sequences and produces the alignment scores fed to the Wilcoxon differential
    test
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Upstream annotation tool that encodes multi-track epigenomic signals into
    the single-track chromatin state sequences EpiAlign aligns
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:b3553f3d8523b337b3a07df41cd43ec69b771f465004c1424251cd41c353a9a6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0495
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# differential-alignment-significance-testing

## Summary

EpiAlign-based vertical local alignment of ChromHMM chromatin state sequences for each protein-coding gene between two epigenomes (e.g. brain E071 vs heart), followed by a two-sample one-sided Wilcoxon rank-sum test on within-group vs cross-group alignment scores to rank differentially aligned, cell-type-specific genes.

## When to use

Use when two Roadmap-style chromatin state annotated epigenomes must be compared gene-by-gene to identify cell-type-specific genes from chromatin state sequence patterns alone (no expression data).

## When NOT to use

- The goal is to compare raw multi-track epigenomic signal matrices directly rather than reduced single-track chromatin state sequences — EpiAlign requires ChromHMM (or similar) state annotation and a compression step first
- The analysis targets horizontal (region-to-region) retrieval of similar 500 kb loci within one epigenome instead of gene-level differential alignment between two tissues
- Genes on chromosome Y must be excluded from the comparison per the article's filtering, so Y-chromosome-only differential questions are out of scope

## Inputs

- Compressed chromatin state sequences for protein-coding gene regions from two Roadmap epigenomes (e.g. E071 brain and a heart sample, ChromHMM state annotations)
- Protein-coding gene coordinates from GENCODE v10 (compatible with the Roadmap database)
- Tissue-specific gene labels (known brain-specific and heart-specific genes) for hypergeometric enrichment validation
- Chromatin state weight definitions (equal or frequency-based weights)

## Outputs

- Per-gene differential alignment P-values ranking all protein-coding genes (top 100 differential genes; STMN4 expected as top hit)
- Within-group and cross-group EpiAlign local alignment scores per gene
- Hypergeometric enrichment counts of tissue-specific genes among top-ranked genes (expected ~9 brain-specific and ~4 heart-specific in top 100, P < 10^-30)
- GO enrichment terms for top-ranked aligned genes (GOrilla, P < 0.0001)

## How to apply

Load compressed chromatin state sequences (consecutive identical ChromHMM states compressed to a single state label) for each protein-coding gene region (GENCODE v10 compatible with Roadmap) in both epigenomes, e.g. brain E071 and heart. Run EpiAlign's modified Smith-Waterman local alignment (dynamic programming with 0-reset, mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, match score = sum of chromatin state weights) on each gene's compressed state sequence within-tissue and across tissues. For each of the 19,935 protein-coding genes, compute a differential P-value with a two-sample one-sided Wilcoxon rank-sum test comparing within-tissue (e.g. brain-vs-brain) to cross-tissue (e.g. brain-vs-heart) alignment score groups. Rank genes by P-value (top hits in the published case: STMN1-like STMN4 first). Assess tissue-specificity enrichment via hypergeometric tests on the top 100 genes and GO enrichment (e.g. GOrilla, P < 0.0001) on the top-ranked aligned genes.

## Related tools

- **EpiAlign** (Performs the modified Smith-Waterman local alignment on compressed chromatin state sequences and produces the alignment scores fed to the Wilcoxon differential test) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Upstream annotation tool that encodes multi-track epigenomic signals into the single-track chromatin state sequences EpiAlign aligns)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign.git
```

## Evaluation signals

- Top-ranked (smallest P-value) gene in the brain-vs-heart differential alignment is STMN4
- Hypergeometric test of known tissue-specific genes among the top 100 ranked genes yields P < 10^-30 (expected ~9 brain-specific and ~4 heart-specific genes)
- GOrilla GO enrichment on top-ranked aligned genes returns terms at P < 0.0001
- Within-tissue vs cross-tissue score groups are computed for every protein-coding gene and tested with a two-sample one-sided Wilcoxon rank-sum test
- Alignment uses the specified penalties: mismatch epsilon_N = 1.5, deletion epsilon_D = 1, and match score = sum of state weights, with consecutive identical states compressed

## Limitations

- Alignment results depend on ChromHMM state annotation quality; the article discusses likely noisy ChromHMM annotations needing biological validation (Supplementary section 12, not provided here)
- Quiescent state sequences (state 'o') contain long uninformative stretches, motivating the compression step; uncompressed sequences of long single states are uninformative for comparison
- No version numbers are stated for ChromHMM or other tools, and no changelog exists for the EpiAlign repository, so exact reproducibility of versions is uncertain

## Evidence

- [methods] EpiAlign then performs a local alignment between two genomic regions based on their compressed state sequences.: "EpiAlign then performs a local alignment between two genomic regions based on their compressed state sequences."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [results] we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [results] we compare the chromatin state sequences for each of the 19,935 protein-coding genes between every pair of samples: "we compare the chromatin state sequences for each of the 19,935 protein-coding genes between every pair of samples"
- [readme] we recommend you to download our package by "git clone" ruther than downloading the zip file.: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
