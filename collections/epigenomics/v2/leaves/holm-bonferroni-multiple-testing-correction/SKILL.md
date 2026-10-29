---
name: holm-bonferroni-multiple-testing-correction
description: Use when many candidate motifs (e.g., DiNAMO lattice nodes scored by
  mutual information) each yield a raw 2x2 contingency-table P-value that must be
  jointly corrected before applying a significance threshold.
when_to_use_negative:
- When only a single hypothesis is being tested — the Holm–Bonferroni correction exists
  precisely because DiNAMO evaluates thousands of lattice motif candidates simultaneously
- During the earlier lattice simplification stage, which deliberately ranks motifs
  by mutual information rather than Fisher's exact test P-value to remain independent
  of occurrence count
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: DiNAMO
  role: 'Implements the final statistical filtering step: computes Fisher''s exact
    test P-values per selected IUPAC motif, applies Holm–Bonferroni adjustment, and
    filters by the ''-t'' threshold into the MEME-format output'
  repo: https://github.com/bonsai-team/DiNAMO
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-018-2215-1
    title: 'DiNAMO: highly sensitive DNA motif discovery in high-throughput sequencing
      data'
  source_content_hash: sha256:7ac6a8faf17da89b56d4d4de4d67fbb0afa1070582c4f9d80d324461545ec432
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bonsai-team/DiNAMO
  tools:
  - DiNAMO
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# holm-bonferroni-multiple-testing-correction

## Summary

Adjusts per-motif Fisher's exact test P-values computed during DiNAMO's statistical filtering stage using the Holm–Bonferroni sequentially rejective method, so that over-represented IUPAC motif calls are not inflated by the multiple-comparison problem across the thousands of candidate motifs in the lattice.

## When to use

Use when many candidate motifs (e.g., DiNAMO lattice nodes scored by mutual information) each yield a raw 2x2 contingency-table P-value that must be jointly corrected before applying a significance threshold.

## When NOT to use

- When only a single hypothesis is being tested — the Holm–Bonferroni correction exists precisely because DiNAMO evaluates thousands of lattice motif candidates simultaneously
- During the earlier lattice simplification stage, which deliberately ranks motifs by mutual information rather than Fisher's exact test P-value to remain independent of occurrence count

## Inputs

- MI-ranked candidate IUPAC motifs with per-motif 2x2 contingency tables (a, b, c, d presence/absence counts in positive signal.fa and negative control.fa datasets)
- Significance threshold f (DiNAMO '-t' parameter, default 0.05)

## Outputs

- Holm–Bonferroni-adjusted Fisher's exact test P-values for all selected motifs
- Set of significantly over-represented motifs passing the adjusted-P threshold, written to overrepresented_motifs.meme in MEME format

## How to apply

After the IUPAC lattice is simplified with mutual information and each selected motif has a per-motif 2x2 contingency table (presence/absence counts a, b, c, d in the positive and negative FASTA datasets), compute a Fisher's exact test P-value for every candidate to test over-representation in the positive dataset. Then rank the P-values and apply the Holm–Bonferroni sequentially rejective adjustment across all tested motifs to counteract the problem of multiple comparisons. Retain only motifs whose adjusted P-value falls below the significance threshold, which defaults to 0.05 in DiNAMO and can be changed via the '-t' parameter. Rationale: MI captures motif–dataset dependency independently of occurrence count, so the exact-test-plus-correction step supplies the rigorous significance control before motifs are written to the MEME-format output file.

## Related tools

- **DiNAMO** (Implements the final statistical filtering step: computes Fisher's exact test P-values per selected IUPAC motif, applies Holm–Bonferroni adjustment, and filters by the '-t' threshold into the MEME-format output) — https://github.com/bonsai-team/DiNAMO

## Examples

```
dinamo -pf signal.fa -nf control.fa -l 6 -t 0.05
```

## Evaluation signals

- Every retained motif has a Holm–Bonferroni-adjusted Fisher's exact test P-value strictly below the 0.05 threshold (or the user-supplied '-t' value)
- The set of adjusted P-values is monotonically non-decreasing relative to the raw P-value ordering (no raw P-value below its Holm-adjusted counterpart)
- The output file overrepresented_motifs.meme parses as valid MEME-format motif records
- Running with a stricter '-t' threshold strictly reduces (never increases) the number of retained over-represented motifs
- No motif with a raw Fisher's exact test P-value failing over-representation in the positive dataset survives the correction

## Limitations

- The correction is applied only to motifs selected from the simplified lattice — candidates already eliminated by the MI-based simplification never reach this significance test, so the family of tests is the post-MI selection
- The Holm–Bonferroni adjustment can be conservative, which is relevant given DiNAMO's stated goal of finding weakly represented motifs implanted at frequencies as low as 0.5% of sequences
- The significance threshold is configurable via '-t', so results are only comparable across runs when the same threshold is used

## Evidence

- [methods] Final statistical filtering rationale: [evidence span withheld — anchor status: not_found]
- [methods] Purpose of the correction: "The Holm–Bonferroni method [25] is applied to adjust the P-values and counteract the problem of multiple comparisons."
- [methods] Why MI, not raw P-value, drives motif selection first: "the MI (mutual information) captures the dependency between a motif and a dataset independently from its number of occurrences."
- [readme] Configurable significance threshold in DiNAMO CLI: "change Fisher's exact test p-value threshold *f* (default: 0.05)"
- [readme] Output format of surviving motifs: "The output file contains the over-represented motifs found in the positive dataset compared to the negative dataset, in the MEME format"
