---
name: sex-specific-epigenomic-differential-analysis
description: Use when comparing chromatin state sequences of protein-coding genes
  across male and female Roadmap epigenomic samples to test whether same-sex pairs
  align better than different-sex pairs and identify sex-linked differentially aligned
  genes.
when_to_use_negative:
- Input includes genes on chromosome Y — the analysis explicitly excludes Y-chromosome
  genes from the male/female comparison.
- The question only needs global epigenome similarity (e.g., mark-overlap or correlation
  summaries) rather than sequential local alignment of compressed chromatin state
  sequences.
- Epigenomic signals have not been reduced to chromatin state sequences (e.g., raw
  ChIP-seq tracks with no ChromHMM segmentation), since EpiAlign operates on discrete
  state label sequences.
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_3301
tools:
- name: EpiAlign
  role: Dynamic-programming local alignment of compressed chromatin state sequences
    between male-male, female-female, and male-female sample pairs to produce the
    three alignment score groups per gene
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into single-track chromatin state sequences
    per sample before alignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:31566e7b5ccaa0ce0a5b319528ff97ddcb04428c2c4ca04af454b83de553790f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# sex-specific-epigenomic-differential-analysis

## Summary

Detect differentially aligned chromatin state sequences between male and female Roadmap epigenomes by running EpiAlign local alignments per gene across male-vs-male, female-vs-female, and male-vs-female sample pairs, then testing score groups with a two-sample one-sided Wilcoxon rank-sum test to surface sex-linked (X-chromosome) differentially aligned genes.

## When to use

Use when comparing chromatin state sequences of protein-coding genes across male and female Roadmap epigenomic samples to test whether same-sex pairs align better than different-sex pairs and identify sex-linked differentially aligned genes.

## When NOT to use

- Input includes genes on chromosome Y — the analysis explicitly excludes Y-chromosome genes from the male/female comparison.
- The question only needs global epigenome similarity (e.g., mark-overlap or correlation summaries) rather than sequential local alignment of compressed chromatin state sequences.
- Epigenomic signals have not been reduced to chromatin state sequences (e.g., raw ChIP-seq tracks with no ChromHMM segmentation), since EpiAlign operates on discrete state label sequences.

## Inputs

- NIH Roadmap epigenomic datasets (release 9, 2804 genome-wide epigenomic datasets across 127 tissue/cell types) split into male and female samples
- ChromHMM chromatin state annotations (single-track state sequences) for each sample
- GENCODE v10 protein-coding gene annotations compatible with the Roadmap database (19,935 genes, Y-chromosome genes excluded)
- EpiAlign scoring parameters: mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, 0 floor in the recurrence

## Outputs

- Three groups of pairwise EpiAlign alignment scores per gene (male-vs-male, female-vs-female, male-vs-female)
- Per-gene two-sample one-sided Wilcoxon rank-sum test P-values
- Ranked top-200 gene table with chromosome annotation (expected: 188/200 X-chromosome genes)
- Results file sex_differential_alignment_results.csv containing score groups, P-values, and the annotated top-200 table

## How to apply

First annotate the male and female NIH Roadmap epigenomic samples with ChromHMM to encode multi-track epigenomic signals into single-track chromatin state sequences. Compress consecutive occurrences of the same chromatin state into a single state label per gene so long quiescent stretches do not dominate alignments. For each of the 19,935 GENCODE v10 protein-coding genes compatible with the Roadmap database (excluding genes on chromosome Y), run the EpiAlign dynamic-programming local alignment (modified Smith-Waterman with a 0 floor; mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1) between every male-male, female-female, and male-female sample pair. For each gene, run a two-sample one-sided Wilcoxon rank-sum test of male-vs-male scores against male-vs-female scores, rank genes by P-value, take the top 200, and count X-chromosome genes. The rationale is that EpiAlign is the first pairwise alignment-based method incorporating sequential chromatin state patterns, so sex-specific chromatin organization shows up as score separation between same-sex and cross-sex comparisons.

## Related tools

- **EpiAlign** (Dynamic-programming local alignment of compressed chromatin state sequences between male-male, female-female, and male-female sample pairs to produce the three alignment score groups per gene) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into single-track chromatin state sequences per sample before alignment)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- The top 200 genes with the smallest Wilcoxon P-values contain ~188 X-chromosome genes, reproducing the reported sex-linked excess.
- Per gene, the male-vs-male alignment score distribution is stochastically higher than the male-vs-female distribution (one-sided Wilcoxon test significance), and female-vs-female scores show the same separation pattern.
- Sequences fed to EpiAlign are compressed: consecutive occurrences of the same chromatin state collapse to a single state label, so no run-length stretches dominate the alignments.
- Alignment recurrence uses a 0 floor (Smith-Waterman style) with epsilon_N = 1.5 mismatch and epsilon_D = 1 deletion penalties; scores are non-negative.
- The output CSV sex_differential_alignment_results.csv contains three score groups, per-gene P-values, and the annotated top-200 table, and the same-sex vs different-sex separation is statistically significant.

## Limitations

- Downloading the package as a zip file may cause errors in links; the README recommends cloning via 'git clone' instead.
- Uncompressed chromatin state sequences contain long stretches of a single (quiescent) state, which would dominate alignments — the compression step is required, and unprocessed sequences are unsuitable inputs.
- The male/female comparison is restricted to protein-coding genes compatible with the Roadmap database and excludes chromosome Y, limiting conclusions to the analyzed gene set.

## Evidence

- [methods] ChromHMM encodes multi-track signals before compression: "We first apply a chromatin state annotation method (e.g. ChromHMM (9)) to encode multi-track epigenomic signals into single-track chromatin state sequences"
- [methods] Compression step rationale: "Second, we compress consecutive occurrences of the same state into a state label."
- [results] Wilcoxon test between score groups: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [results] Y-chromosome exclusion filter: "with the exception of genes on chromosome Y"
- [readme] README install warning: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
