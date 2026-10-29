---
name: contingency-table-probability-estimation
description: Use when a set of candidate motifs (or features) has already been ranked
  by a score such as mutual information and each candidate has a 2x2 contingency table
  of counts in a positive and a negative dataset that must be tested for over-representation
  with multiple-comparison control.
when_to_use_negative:
- You need an occurrence-count-independent comparison of motifs during candidate selection
  — the article states MI is used for that stage because 'MI captures the dependency
  between a motif and a dataset independently from its number of occurrences'; the
  Fisher's exact test is reserved for the final significance step.
- Your input is a single dataset without a negative/control dataset, so no 2x2 contingency
  table (positive vs negative presence/absence counts) can be constructed.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3511
tools:
- name: DiNAMO
  role: Computes the Fisher's exact test P-value from each motif's 2x2 contingency
    table, applies the Holm–Bonferroni adjustment, filters motifs at the '-t' threshold
    (default 0.05), and writes survivors in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:29ec85718cd3be8f3f1bda861ef9bc0eadf39333b872bad48c2d9d5ed0800d58
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# contingency-table-probability-estimation

## Summary

Assess the statistical significance of candidate IUPAC motifs by computing a Fisher's exact test P-value from each motif's 2x2 contingency table (presence/absence counts a, b, c, d in positive vs negative datasets) and then adjusting all P-values with the Holm–Bonferroni method to counteract multiple comparisons. This is DiNAMO's final filtering step after mutual-information-based motif selection.

## When to use

Use when a set of candidate motifs (or features) has already been ranked by a score such as mutual information and each candidate has a 2x2 contingency table of counts in a positive and a negative dataset that must be tested for over-representation with multiple-comparison control.

## When NOT to use

- You need an occurrence-count-independent comparison of motifs during candidate selection — the article states MI is used for that stage because 'MI captures the dependency between a motif and a dataset independently from its number of occurrences'; the Fisher's exact test is reserved for the final significance step.
- Your input is a single dataset without a negative/control dataset, so no 2x2 contingency table (positive vs negative presence/absence counts) can be constructed.

## Inputs

- MI-ranked IUPAC motif candidates with per-motif 2x2 contingency tables (counts a, b, c, d) from the positive and negative datasets
- Positive dataset FASTA file (e.g. signal.fa)
- Negative dataset FASTA file (e.g. control.fa)
- Significance threshold (default 0.05, configurable via DiNAMO's '-t' parameter)

## Outputs

- Holm–Bonferroni-adjusted Fisher's exact test P-value for each selected motif
- Set of significantly over-represented motifs (adjusted P-value < 0.05 by default) written in MEME format (overrepresented_motifs.meme)

## How to apply

Load the MI-ranked motif candidates together with their per-motif 2x2 contingency tables (presence/absence counts a, b, c, d in the positive and negative datasets) produced by the lattice-exploration stage. For each candidate motif, compute the Fisher's exact test P-value from its contingency table to assess over-representation in the positive dataset; DiNAMO uses MI for selection because MI captures the dependency between a motif and a dataset independently from its number of occurrences, while the Fisher's exact test supplies the final significance measure. Apply the Holm–Bonferroni sequentially rejective adjustment to all computed P-values to counteract the multiple-comparison problem. Retain only motifs whose adjusted P-value is below the significance threshold, which defaults to 0.05 in DiNAMO and is configurable via the '-t' parameter. Write the surviving significantly over-represented motifs to the output file in MEME format.

## Related tools

- **DiNAMO** (Computes the Fisher's exact test P-value from each motif's 2x2 contingency table, applies the Holm–Bonferroni adjustment, filters motifs at the '-t' threshold (default 0.05), and writes survivors in MEME format) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -t 0.05
```

## Evaluation signals

- Every retained motif has a Holm–Bonferroni-adjusted Fisher's exact test P-value strictly below the 0.05 threshold (or the user-specified '-t' threshold).
- Each tested motif has a well-formed 2x2 contingency table (a, b, c, d presence/absence counts) covering both the positive and negative datasets.
- The output file overrepresented_motifs.meme parses as valid MEME-format motif records (per meme-suite.org format).
- No candidate motif that was eliminated during the MI lattice-simplification stage is re-tested; the Fisher's exact test runs only on motifs selected by MI.

## Limitations

- Fisher's exact test P-values across many candidate motifs suffer from the multiple-comparison problem, which is why Holm–Bonferroni adjustment is mandatory in this workflow.
- The statistical test is applied only to motifs that survived MI-based lattice simplification; it is not used for motif ranking or selection itself.
- The adjustable threshold '-t' changes the stringency; deviating from the 0.05 default should be justified for the target false-discovery rate.

## Evidence

- [methods] Fisher's exact test on contingency tables: [evidence span withheld — anchor status: not_found]
- [methods] Holm–Bonferroni rationale: "The Holm–Bonferroni method [25] is applied to adjust the P-values and counteract the problem of multiple comparisons."
- [methods] Why MI precedes the test: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [methods] Contingency table source: "each vertex of the IUPAC lattice has a contingency table that contains the counts for the files"
- [readme] Default threshold and parameter: "change Fisher's exact test p-value threshold *f* (default: 0.05)"
