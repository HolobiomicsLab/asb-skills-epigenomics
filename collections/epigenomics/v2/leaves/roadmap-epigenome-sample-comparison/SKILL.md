---
name: roadmap-epigenome-sample-comparison
description: Use when comparing epigenomic samples (e.g. Roadmap ESC/heart/brain)
  via ChromHMM chromatin state sequences and needing an order-aware alignment score
  benchmarked against a frequency-only baseline.
when_to_use_negative:
- The comparison target is gene-region retrieval or motif discovery (e.g. finding
  NRG3-GRIA1-like best matches), not whole-sample group separation
- Chromatin state annotations are absent — samples must already be encoded as single-track
  state sequences via a method such as ChromHMM
- Sequential order of states is irrelevant to the question and a frequency-vector
  comparison is sufficient on its own
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3656
tools:
- name: EpiAlign
  role: Performs the modified Smith-Waterman local alignment of compressed chromatin
    state sequences to yield pairwise alignment scores M_n1,n2
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into single-track chromatin state sequences
    prior to compression and alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:9812d6bf5f94c5cc8c50a4eeedd9b1734a18c81ae688ea8684bef83c0e7a4be1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3656
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# roadmap-epigenome-sample-comparison

## Summary

Reproduce EpiAlign's superiority over a naive chromatin-state frequency baseline by scoring pairwise chromatin-state sequence alignments across Roadmap epigenomes and testing group separation with a one-sided Wilcoxon rank-sum test.

## When to use

Use when comparing epigenomic samples (e.g. Roadmap ESC/heart/brain) via ChromHMM chromatin state sequences and needing an order-aware alignment score benchmarked against a frequency-only baseline.

## When NOT to use

- The comparison target is gene-region retrieval or motif discovery (e.g. finding NRG3-GRIA1-like best matches), not whole-sample group separation
- Chromatin state annotations are absent — samples must already be encoded as single-track state sequences via a method such as ChromHMM
- Sequential order of states is irrelevant to the question and a frequency-vector comparison is sufficient on its own

## Inputs

- Roadmap epigenome chromatin state sequences (ChromHMM-annotated, run-length compressed) per sample
- Sample metadata defining biologically related (male-vs-male) and unrelated (male-vs-female) pair groupings
- EpiAlign penalty parameters eps_N = 1.5 and eps_D = 1

## Outputs

- epialign_vs_frequency_baseline_scores.csv with one row per sample pair (EpiAlign score, frequency-baseline score, group label)
- Two-sample one-sided Wilcoxon rank-sum test statistics and P-values for both score types
- Summary judgment of whether EpiAlign separates related from unrelated pairs at least as strongly as the frequency baseline

## How to apply

Run ChromHMM annotation on Roadmap epigenomic signals and run-length compress the state sequences (collapse consecutive identical states into one label). For each sample pair, compute the naive baseline by building a chromatin state frequency vector per epigenome and scoring similarity from these vectors, ignoring sequential order; then run the EpiAlign dynamic-programming local alignment with match function MF = W(k,S1)+W(l,S2), mismatch NF = -eps_N*(W(k,S1)+W(l,S2)), and deletion DF = -eps_D*W(k,S) using eps_N = 1.5 and eps_D = 1, recording the local alignment score M_n1,n2. Group scores into biologically related (male-vs-male) and unrelated (male-vs-female) pairs and apply the two-sample one-sided Wilcoxon rank-sum test to each score type. Emit a per-pair table with both scores and group labels plus Wilcoxon statistics and P-values (e.g. epialign_vs_frequency_baseline_scores.csv). Rationale: EpiAlign incorporates sequential information of chromatin states, which frequency comparisons discard.

## Related tools

- **EpiAlign** (Performs the modified Smith-Waterman local alignment of compressed chromatin state sequences to yield pairwise alignment scores M_n1,n2) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into single-track chromatin state sequences prior to compression and alignment)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- The one-sided Wilcoxon test on EpiAlign scores gives a significantly higher distribution for male-vs-male than male-vs-female pairs (P < 0.05)
- EpiAlign's group separation (Wilcoxon statistic/P-value) is at least as strong as the frequency-based baseline's, matching the paper's qualitative claim
- Every sample pair has both score types and a valid group label present in the output CSV (no missing rows)
- Compressed sequences contain no consecutive identical state labels (run-length compression verified)
- Alignment runs used the documented penalties eps_N = 1.5 and eps_D = 1

## Limitations

- ChromHMM annotations can be noisy and may need further biological validation (per the paper's discussion of likely noisy annotations, Supplementary section 12)
- No version numbers are stated for ChromHMM or EpiAlign, so exact reproduction may vary
- The paper notes that constructing specialized substitution matrices for chromatin states remains an open research question

## Evidence

- [abstract] EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states.: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
- [methods] we compress consecutive occurrences of the same state into a state label: "Second, we compress consecutive occurrences of the same state into a state label."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [results] we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [readme] README advises downloading the package by git clone rather than zip due to link errors: "we recommend you to download our package by "git clone" ruther than downloading the zip file."
