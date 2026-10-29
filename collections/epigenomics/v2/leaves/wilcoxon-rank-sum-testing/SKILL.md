---
name: wilcoxon-rank-sum-testing
description: Use when you have two independent groups of non-normal, possibly unequal-variance
  scores — such as pairwise alignment scores per gene — and need a per-item P-value
  to rank differential behavior.
when_to_use_negative:
- The two score groups are paired or dependent observations rather than independent
  samples (use a paired test instead).
- The goal is to compare score similarity while ignoring order — e.g., the naive chromatin-state
  frequency baseline — where a rank test on frequency-vector similarity, not sequential
  alignment scores, is the wrong object to test for EpiAlign superiority claims.
- Scores come from a single group with no comparison condition (e.g., horizontal within-epigenome
  query-to-randomized alignment normalization), where no two-sample hypothesis test
  applies.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: EpiAlign
  role: Produces the pairwise local alignment score distributions (male-vs-male, male-vs-female,
    female-vs-female) that the Wilcoxon rank-sum test compares per gene
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track Roadmap epigenomic signals into the single-track chromatin
    state sequences from which the compared alignment scores are derived
  repo: https://github.com/zzz3639/EpiAlign
provenance:
  source_task_ids:
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:ebc0c173b9666e11bc9858ff61bcb3bc747a2f523d8fe24b3e4d94e7561754b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# wilcoxon-rank-sum-testing

## Summary

Apply the two-sample one-sided Wilcoxon rank-sum test to decide whether two independent groups of pairwise scores (e.g., EpiAlign local alignment scores for male-vs-male versus male-vs-female Roadmap epigenome pairs) come from distributions with a shifted location. In the EpiAlign workflow it provides a per-gene, non-parametric P-value that ranks differentially aligned genes without assuming normality.

## When to use

Use when you have two independent groups of non-normal, possibly unequal-variance scores — such as pairwise alignment scores per gene — and need a per-item P-value to rank differential behavior.

## When NOT to use

- The two score groups are paired or dependent observations rather than independent samples (use a paired test instead).
- The goal is to compare score similarity while ignoring order — e.g., the naive chromatin-state frequency baseline — where a rank test on frequency-vector similarity, not sequential alignment scores, is the wrong object to test for EpiAlign superiority claims.
- Scores come from a single group with no comparison condition (e.g., horizontal within-epigenome query-to-randomized alignment normalization), where no two-sample hypothesis test applies.

## Inputs

- Two independent vectors of numeric scores per testing unit (e.g., male-vs-male and male-vs-female EpiAlign local alignment scores for each protein-coding gene)
- Gene/sample metadata linking each score group to chromosomal location (e.g., GENCODE v10 annotations to identify X-chromosome genes)

## Outputs

- Per-gene two-sample one-sided Wilcoxon rank-sum statistics and P-values (e.g., sex_differential_alignment_results.csv)
- Ranked table of top candidate genes by smallest P-value with chromosomal annotation (e.g., top-200 table used for X-chromosome enrichment counting)

## How to apply

Group the pairwise scores per item: in the EpiAlign sex-linked analysis, EpiAlign local alignment scores (computed with the modified Smith-Waterman recurrence, mismatch penalty epsilon_N = 1.5 and deletion penalty epsilon_D = 1) are grouped into male-vs-male and male-vs-female sets for each of the 19,935 protein-coding genes (Y-chromosome genes excluded). For each gene, run a two-sample one-sided Wilcoxon rank-sum test comparing male-vs-male scores against male-vs-female scores, choosing the one-sided alternative that reflects the hypothesis that same-sex pairs score higher. Rank all genes by their P-values, extract the top candidates (the paper uses the top 200 smallest P-values), and test biological enrichment — e.g., counting X-chromosome genes, where 188/200 were on chromosome X. The rank-sum test is chosen because alignment score distributions need not be normal and sample sizes per group are small and unbalanced across epigenome pairs.

## Related tools

- **EpiAlign** (Produces the pairwise local alignment score distributions (male-vs-male, male-vs-female, female-vs-female) that the Wilcoxon rank-sum test compares per gene) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track Roadmap epigenomic signals into the single-track chromatin state sequences from which the compared alignment scores are derived) — https://github.com/zzz3639/EpiAlign

## Examples

```
Rscript -e 'w <- wilcox.test(male_male_scores, male_female_scores, alternative="greater"); cat(w$p.value, "\n")'
```

## Evaluation signals

- Per-gene P-values exist for all tested units (19,935 protein-coding genes after excluding Y-chromosome genes) and fall in [0, 1].
- The one-sided test direction matches the hypothesis: same-sex (male-vs-male) scores are the alternative-greater group versus male-vs-female scores, and the overall separation is significant (P < 0.05) for EpiAlign scores.
- Top-ranked genes are biologically coherent: the top 200 smallest P-values contain 188 X-chromosome genes, matching the reported 188/200 excess.
- Three score groups (male-vs-male, male-vs-female, female-vs-female) are written out separately so same-sex versus different-sex separation can be inspected before testing.
- Results are persisted with per-gene P-values, three score groups, and the annotated top-200 gene table (sex_differential_alignment_results.csv).

## Limitations

- The test is rank-based and does not quantify effect size; the paper reports the 188/200 X-chromosome enrichment count rather than a fold-change from the test itself.
- One gene-level test at a time means the top 200 P-values are subject to multiple-testing pressure; the article handles this by ranking and enrichment analysis rather than formal correction in the cited workflow.
- Interpretation depends on upstream EpiAlign choices (compression of consecutive states, penalty parameters epsilon_N = 1.5 and epsilon_D = 1); changing these changes the score distributions being tested.
- Genes on chromosome Y are excluded from the comparison, so results do not speak to Y-linked differential alignment.

## Evidence

- [results] per-gene Wilcoxon testing in the sex-linked analysis: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [results] top-200 ranking and chromosome X enrichment: "We next perform the Gene Ontology (GO) enrichment analysis (30) on the top 200 genes that receive the smallest P-values"
- [readme] obtaining the EpiAlign package the scores come from: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
