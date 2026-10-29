---
name: gene-level-alignment-score-aggregation
description: Use when chromatin state sequences of thousands of protein-coding genes
  must be compared across two sample groups (e.g. male vs female NIH Roadmap epigenomes)
  to produce per-gene groups of EpiAlign alignment scores and a ranked list of differentially
  aligned genes.
when_to_use_negative:
- The goal is motif discovery within a single epigenome rather than gene-level comparison
  of alignment score distributions across sample groups
- Genes of interest are on chromosome Y — the paper's sex-differential analysis explicitly
  excludes genes on chromosome Y
- Input is raw multi-track epigenomic signal (e.g. ChIP-seq bigWig tracks) rather
  than already-annotated, compressed chromatin state sequences
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3407
- http://edamontology.org/topic_0622
tools:
- name: EpiAlign
  role: Performs the dynamic-programming local alignment of compressed chromatin state
    sequences to produce pairwise alignment scores for each gene between sample pairs
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track Roadmap epigenomic signals into the single-track chromatin
    state sequences that EpiAlign aligns
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:17396dabdc1037ee0706f0f669eb916ce61516c57c08a12613bc848c1fd63a80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3407
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gene-level-alignment-score-aggregation

## Summary

Aggregates pairwise EpiAlign local alignment scores of chromatin state sequences at the gene level across sample groups (e.g. male-vs-male, male-vs-female, female-vs-female Roadmap epigenomes), then ranks genes by two-sample one-sided Wilcoxon P-values to identify differentially aligned, sex-linked genes.

## When to use

Use when chromatin state sequences of thousands of protein-coding genes must be compared across two sample groups (e.g. male vs female NIH Roadmap epigenomes) to produce per-gene groups of EpiAlign alignment scores and a ranked list of differentially aligned genes.

## When NOT to use

- The goal is motif discovery within a single epigenome rather than gene-level comparison of alignment score distributions across sample groups
- Genes of interest are on chromosome Y — the paper's sex-differential analysis explicitly excludes genes on chromosome Y
- Input is raw multi-track epigenomic signal (e.g. ChIP-seq bigWig tracks) rather than already-annotated, compressed chromatin state sequences

## Inputs

- ChromHMM chromatin state sequences (single-track state labels) per gene per epigenomic sample from the NIH Roadmap Epigenomics project (release 9)
- Sample metadata partitioning Roadmap epigenomes into male and female groups
- Gene coordinates / gene list for 19,935 GENCODE v10 protein-coding genes compatible with the Roadmap database

## Outputs

- Three groups of pairwise EpiAlign local alignment scores per gene (male-vs-male, female-vs-female, male-vs-female)
- Per-gene two-sample one-sided Wilcoxon rank-sum test P-values
- Annotated table of the top 200 genes with smallest P-values, including chromosome assignment (e.g. results written to sex_differential_alignment_results.csv)

## How to apply

First encode multi-track epigenomic signals with ChromHMM into single-track chromatin state sequences, then compress consecutive occurrences of the same state into a single state label so long quiescent stretches do not dominate alignments. For each of the 19,935 GENCODE v10 protein-coding genes compatible with the Roadmap database (excluding genes on chromosome Y), run the EpiAlign dynamic-programming local alignment (modified Smith-Waterman with a 0 floor, mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1) between every sample pair within and across the male/female groups, yielding three groups of alignment scores per gene. For each gene, perform a two-sample one-sided Wilcoxon rank-sum test comparing male-vs-male against male-vs-female scores. Rank genes by P-value, extract the top 200, and annotate them by chromosome to quantify sex-linkage enrichment. Decision point: the compression step is essential when sequences contain long single-state runs; frequency-based vs equal state weights can be chosen per the paper's two weighting schemes.

## Related tools

- **EpiAlign** (Performs the dynamic-programming local alignment of compressed chromatin state sequences to produce pairwise alignment scores for each gene between sample pairs) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track Roadmap epigenomic signals into the single-track chromatin state sequences that EpiAlign aligns)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- Three distinct score groups per gene are produced (male-vs-male, female-vs-female, male-vs-female) for all 19,935 protein-coding genes
- Same-sex comparison scores separate from different-sex scores: the two-sample one-sided Wilcoxon test is statistically significant for differentially aligned genes
- Among the top 200 genes ranked by smallest Wilcoxon P-value, 188 reside on chromosome X (the paper's reported excess of sex-linked genes)
- Compression step applied: consecutive identical chromatin states are collapsed to a single state label before alignment
- Genes on chromosome Y are absent from the analysis, matching the paper's exclusion filter

## Limitations

- Downloading the EpiAlign zip file directly may cause errors in links; the maintainers recommend git clone instead
- Alignment results depend on ChromHMM annotations; the paper notes noisy ChromHMM annotations may require further biological validation
- Long uncompressed stretches of a single (quiescent) state can dominate alignments if the compression step is skipped
- Only protein-coding genes compatible with the Roadmap database and excluding chromosome Y are covered

## Evidence

- [methods] chromatin state annotation to encode multi-track epigenomic signals into single-track chromatin state sequences: "We first apply a chromatin state annotation method (e.g. ChromHMM (9)) to encode multi-track epigenomic signals into single-track chromatin state sequences"
- [methods] compression step: "Second, we compress consecutive occurrences of the same state into a state label."
- [results] per-gene pairwise comparison: "we compare the chromatin state sequences for each of the 19,935 protein-coding genes between every pair of samples"
- [results] Wilcoxon test per gene: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [results] top 200 gene analysis: "We next perform the Gene Ontology (GO) enrichment analysis (30) on the top 200 genes that receive the smallest P-values"
- [results] chromosome Y exclusion: "with the exception of genes on chromosome Y"
- [readme] installation caveat: "Downloading zip file directly may cause errors in links."
