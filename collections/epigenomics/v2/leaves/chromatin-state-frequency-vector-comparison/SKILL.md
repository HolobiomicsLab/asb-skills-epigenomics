---
name: chromatin-state-frequency-vector-comparison
description: Use when benchmarking an order-aware chromatin-state comparison method
  (e.g. EpiAlign) against a simple, order-agnostic baseline, or when only overall
  chromatin state composition — not sequence order — is needed to distinguish epigenomic
  samples.
when_to_use_negative:
- The analysis goal is to retrieve genomic regions similar to a query region in terms
  of epigenomic signal order — the frequency baseline ignores sequential information,
  which the EpiAlign paper identifies as the gap in former epigenome comparative studies
- Chromatin state sequences have not yet been annotated and compressed (ChromHMM states
  + run-length compression); raw multi-track epigenomic signal matrices are not the
  direct input
- 'A principled similarity measure is needed on its own, not a deliberately naive
  comparator: this skill exists to be the weak baseline that EpiAlign is shown to
  outperform'
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: EpiAlign
  role: Order-aware dynamic-programming local alignment method run on the identical
    sample pairs to provide the comparison scores that the frequency-vector baseline
    is benchmarked against
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Hidden Markov model that assigns discrete chromatin state labels to genomic
    regions from multi-track epigenetic marks, producing the sequences that are reduced
    to frequency vectors
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:33e4c185b570012a2a137257f4f999e896129b98fb4f6700ab94c0ae4d516f11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatin-state-frequency-vector-comparison

## Summary

A naive baseline for comparing Roadmap epigenomes that represents each sample's chromatin state sequence as a vector of state frequency proportions and scores pairwise similarity from these vectors, deliberately ignoring sequential order. It serves as the negative/naive comparator used to demonstrate the superiority of EpiAlign's order-aware local alignment.

## When to use

Use when benchmarking an order-aware chromatin-state comparison method (e.g. EpiAlign) against a simple, order-agnostic baseline, or when only overall chromatin state composition — not sequence order — is needed to distinguish epigenomic samples.

## When NOT to use

- The analysis goal is to retrieve genomic regions similar to a query region in terms of epigenomic signal order — the frequency baseline ignores sequential information, which the EpiAlign paper identifies as the gap in former epigenome comparative studies
- Chromatin state sequences have not yet been annotated and compressed (ChromHMM states + run-length compression); raw multi-track epigenomic signal matrices are not the direct input
- A principled similarity measure is needed on its own, not a deliberately naive comparator: this skill exists to be the weak baseline that EpiAlign is shown to outperform

## Inputs

- Compressed chromatin state sequences per Roadmap epigenome sample (ChromHMM annotations with consecutive identical states collapsed into single state labels)
- Sample metadata needed to define biologically related vs unrelated pairs (e.g. sex/tissue labels for male-vs-male vs male-vs-female grouping)

## Outputs

- Per-sample-pair frequency-based similarity scores paired with EpiAlign local alignment scores (M_n1,n2)
- Two-sample one-sided Wilcoxon rank-sum test statistics and P-values comparing score distributions for related vs unrelated pairs, for both methods
- Summary table, e.g. epialign_vs_frequency_baseline_scores.csv, with one row per sample pair containing both score types and group labels

## How to apply

Load the compressed chromatin state sequences produced by ChromHMM annotation followed by run-length compression (collapsing consecutive identical states into single labels, per EpiAlign preprocessing). For every sample pair, represent each epigenome's chromatin states as a frequency vector and compute the pair similarity score from these vectors, intentionally discarding sequential information — this is the key distinction from EpiAlign's dynamic-programming local alignment with MF = W(k,S1)+W(l,S2), NF = -eps_N*(W(k,S1)+W(l,S2)), and DF = -eps_D*W(k,S) (eps_N = 1.5, eps_D = 1). Group the resulting scores into biologically related (male-vs-male) and unrelated (male-vs-female) pairs, then apply a two-sample one-sided Wilcoxon rank-sum test to each score type to quantify how well each method separates the groups. Assemble a per-pair table (e.g. epialign_vs_frequency_baseline_scores.csv) with both score types and group labels plus the Wilcoxon statistics and P-values, so the baseline can be directly compared to EpiAlign on detecting annotated genes and tissue-associated genes among top-aligned regions across the 16 ESC, heart, and brain samples.

## Related tools

- **EpiAlign** (Order-aware dynamic-programming local alignment method run on the identical sample pairs to provide the comparison scores that the frequency-vector baseline is benchmarked against) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Hidden Markov model that assigns discrete chromatin state labels to genomic regions from multi-track epigenetic marks, producing the sequences that are reduced to frequency vectors)

## Evaluation signals

- One-sided Wilcoxon rank-sum test on frequency-baseline scores for male-vs-male vs male-vs-female pairs yields P < 0.05, confirming the baseline has some discriminating power
- The baseline's group separation is weaker than (or at best equal to) EpiAlign's, reproducing the paper's qualitative claim that EpiAlign better distinguishes epigenomic samples
- Frequency vectors are computed on compressed state sequences (consecutive identical states collapsed), matching EpiAlign's preprocessing so the comparison is fair
- Scores are computed for the same sample pairs used for EpiAlign, and the summary table (epialign_vs_frequency_baseline_scores.csv) contains both score types plus group labels for every pair
- EpiAlign outperforms the baseline in detecting annotated genes and tissue-associated genes among top-aligned regions across the 16 ESC, heart, and brain samples

## Limitations

- Ignores sequential information of chromatin states — the exact weakness the EpiAlign paper identifies in former comparative studies, which is why it loses to EpiAlign on tissue-associated gene detection
- All-chromosomes frequency vectors treat common states like quiescent stretches equally with rare, informative states; the article's frequency-based weighting scheme (larger weights for less common states) is a refinement, but the naive vector baseline uses raw proportions
- Results are tied to the ChromHMM annotation quality; the discussion notes likely noisy ChromHMM annotations that need further biological validation

## Evidence

- [intro] Gap: previous epigenome comparative studies failed to incorporate the sequential information of chromatin states.: "former epigenome comparative studies failed to effectively incorporate the sequential information of chromatin states"
- [intro] ChromHMM assigns discrete chromatin state labels to genomic regions based on signals of multiple epigenetic marks using a hidden Markov model.: "ChromHMM assigns discrete chromatin state labels to genomic regions based on signals of multiple epigenetic marks using a hidden Markov model (9)."
- [methods] Compression step collapses consecutive identical states, motivating the preprocessing shared by both the baseline and EpiAlign.: "The motivation of adding a compression step lies in the fact that most uncompressed (chromatin state) sequences contain long stretches of a single chromatin state"
- [results] Two-sample one-sided Wilcoxon test used to compare score groups across sample pairs.: "we perform the two-sample one-sided Wilcoxon test between male-vs-male scores and male-vs-female scores for each protein-coding gene"
- [abstract] EpiAlign can group and distinguish epigenomic samples — the capability the frequency baseline is benchmarked against.: "EpiAlign can also detect common chromatin state patterns across multiple epigenomes, and it will serve as a useful tool to group and distinguish epigenomic samples based on genome-wide or local"
