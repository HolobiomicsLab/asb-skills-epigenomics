---
name: fisher-exact-test-motif-significance
description: Use when you have DiNAMO MI-ranked IUPAC motif candidates with per-motif
  contingency tables (presence/absence counts in signal and control FASTA datasets)
  and need significance-filtered over-represented motifs.
when_to_use_negative:
- Motif candidates have not yet been scored and selected by the mutual-information
  lattice simplification step, so contingency tables are not available
- The analysis requires position-specific PWMs from scratch (e.g. EM-based MEME discovery)
  rather than exact degenerate IUPAC motif discovery against a control dataset
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0203
tools:
- name: DiNAMO
  role: Computes Fisher's exact test P-values per motif, applies Holm–Bonferroni adjustment
    at the '-t' threshold, and writes surviving over-represented motifs in MEME format
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:27137cfb361c1bd8a5c81c32de120484d4c687d96ccc78465dc1436e7b57ceea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# fisher-exact-test-motif-significance

## Summary

Assign statistical significance to candidate IUPAC motifs discovered by DiNAMO by computing a Fisher's exact test P-value from each motif's 2x2 contingency table (positive vs. negative dataset) and adjusting with the Holm–Bonferroni method, retaining only over-represented motifs at the 0.05 threshold.

## When to use

Use when you have DiNAMO MI-ranked IUPAC motif candidates with per-motif contingency tables (presence/absence counts in signal and control FASTA datasets) and need significance-filtered over-represented motifs.

## When NOT to use

- Motif candidates have not yet been scored and selected by the mutual-information lattice simplification step, so contingency tables are not available
- The analysis requires position-specific PWMs from scratch (e.g. EM-based MEME discovery) rather than exact degenerate IUPAC motif discovery against a control dataset

## Inputs

- MI-ranked IUPAC motif candidates with per-motif 2x2 contingency tables (a, b, c, d counts) from DiNAMO's lattice-exploration stage
- Positive dataset in multi-FASTA format (e.g. signal.fa, GATA1 ChIP-seq peaks)
- Negative control dataset in multi-FASTA format (e.g. control.fa)
- Significance threshold (default 0.05, settable with DiNAMO '-t <f>')

## Outputs

- Set of significantly over-represented IUPAC motifs with Holm–Bonferroni-adjusted Fisher's exact test P-values below 0.05
- MEME-format motif output file (e.g. overrepresented_motifs.meme) parseable by the MEME suite

## How to apply

Load the MI-ranked motif candidates together with their 2x2 contingency tables (presence/absence counts a, b, c, d for positive signal.fa and negative control.fa datasets) produced by DiNAMO's lattice-exploration stage. For each candidate motif, compute the Fisher's exact test P-value from its contingency table to assess over-representation in the positive dataset, then apply the Holm–Bonferroni sequentially rejective adjustment to all P-values to counteract the multiple-comparison problem. Filter motifs to keep only those whose adjusted P-value falls below the significance threshold, which defaults to 0.05 and is configurable via DiNAMO's '-t <f>' parameter. The rationale is that mutual information ranks motifs independently of occurrence count, so the Fisher's exact test provides the final discriminative significance check on occurrence counts before writing surviving motifs to a MEME-format output file.

## Related tools

- **DiNAMO** (Computes Fisher's exact test P-values per motif, applies Holm–Bonferroni adjustment at the '-t' threshold, and writes surviving over-represented motifs in MEME format) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -t 0.05 -o overrepresented_motifs.meme
```

## Evaluation signals

- Every retained motif has a Holm–Bonferroni-adjusted Fisher's exact test P-value strictly below the 0.05 threshold (or the value set via '-t')
- Output file parses as valid MEME-format motif records
- The top recovered motif matches the expected transcription factor motif, verifiable against the corresponding JASPAR frequency matrix as done in the paper
- No motif appears in the output whose raw Fisher P-value ranking, after Holm–Bonferroni adjustment, should have been rejected under the sequentially rejective procedure
- Motif occurrences in the positive dataset exceed those expected by chance relative to the negative control dataset

## Limitations

- Fisher's exact test P-value depends on occurrence count, unlike MI, which captures motif–dataset dependency independently from the number of occurrences; motifs filtered here may include weakly represented motifs that MI favored
- The Holm–Bonferroni adjustment is conservative under very large numbers of candidate motifs, potentially missing weakly over-represented secondary motifs
- Default threshold 0.05 must be explicitly changed via '-t' if a different significance level is desired

## Evidence

- [methods] Statistical filtering with Fisher's exact test: [evidence span withheld — anchor status: not_found]
- [methods] Holm–Bonferroni purpose: "The Holm–Bonferroni method [25] is applied to adjust the P-values and counteract the problem of multiple comparisons."
- [readme] Threshold parameter: "* change Fisher's exact test p-value threshold *f* (default: 0.05)

        -t <f>"
- [readme] Output format: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
- [methods] MI vs Fisher rationale: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
