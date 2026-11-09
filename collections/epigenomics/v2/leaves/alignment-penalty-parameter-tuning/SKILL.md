---
name: alignment-penalty-parameter-tuning
description: Use when configuring EpiAlign's mismatch/deletion penalties (epsilon_N=1.5,
  epsilon_D=1 defaults) or choosing equal vs frequency-based chromatin state weights
  W(k,S) before scoring local alignments of compressed state sequences.
when_to_use_negative:
- When the chromatin state sequences have not been compressed — the paper notes most
  uncompressed sequences contain long stretches of a single chromatin state, which
  distorts the state weights that scale the penalties.
- When the goal is an explicit substitution matrix between chromatin states — the
  paper leaves constructing more specialized substitution matrices as an important
  future research question, outside what this penalty tuning covers.
- When the input is raw multi-track epigenomic signal matrices rather than single-track
  state sequences produced by an annotation method such as ChromHMM.
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: EpiAlign
  role: Provides the dynamic-programming local alignment whose mismatch, N-gap, and
    deletion score functions (NF, DF with epsilon_N/epsilon_D) this skill configures
    and validates
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Produces the single-track chromatin state label sequences whose state frequencies
    feed the frequency-based weights W(k,S) used in penalty scaling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:f9e7c54f5f3237b3e0c22125e79f1c0c1d3cb537cb14b2284a635784a0d1e3ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# alignment-penalty-parameter-tuning

## Summary

EpiAlign's local-alignment scores are driven by weight-scaled penalties — mismatch/N-gap NF = -epsilon_N*(W(k,S1)+W(l,S2)) and deletion DF(k,S) = -epsilon_D*W(k,S), with defaults epsilon_N=1.5 and epsilon_D=1 — combined with a chromatin state-weight scheme (equal W=1 vs frequency-based weights favoring rarer states).

## When to use

Use when configuring EpiAlign's mismatch/deletion penalties (epsilon_N=1.5, epsilon_D=1 defaults) or choosing equal vs frequency-based chromatin state weights W(k,S) before scoring local alignments of compressed state sequences.

## When NOT to use

- When the chromatin state sequences have not been compressed — the paper notes most uncompressed sequences contain long stretches of a single chromatin state, which distorts the state weights that scale the penalties.
- When the goal is an explicit substitution matrix between chromatin states — the paper leaves constructing more specialized substitution matrices as an important future research question, outside what this penalty tuning covers.
- When the input is raw multi-track epigenomic signal matrices rather than single-track state sequences produced by an annotation method such as ChromHMM.

## Inputs

- Compressed chromatin state sequences (compressed_sequences.txt) derived from ChromHMM-style single-track state annotations
- Penalty settings: epsilon_N (default 1.5) and epsilon_D (default 1), plus a weight mode (equal weights vs frequency-based W(k,S))

## Outputs

- Final local-alignment score M[n1,n2] per region pair (alignment_scores.csv)
- Alignment path as triplets (e.g., F=mmd1m, U1=1234, U2=1203 for the S1=abca vs S2=aba validation example)

## How to apply

After the compression step collapses consecutive identical state labels (abbcc -> abc), choose a weight scheme: equal weights set W(k,S)=1 for every state in sequence S, while frequency-based weights assign larger weights to less common chromatin states (detailed in Supplementary section 1). Define the scoring so matches add MF(k,l,S1,S2)=W(k,S1)+W(l,S2), mismatches and N-gaps subtract NF=-epsilon_N*(W(k,S1)+W(l,S2)), and deletions subtract DF(k,S)=-epsilon_D*W(k,S), starting from the paper defaults epsilon_N=1.5 and epsilon_D=1. Because penalties scale with the summed state weights, switching to frequency-based weights raises the effective penalty for rarer states, so recompute weights whenever the underlying ChromHMM annotation or sample set changes. Run the local-alignment DP recurrence (Equation 4), which includes 0 in the max, initializes the first row/column to zero, and reports M[n1,n2] as the final alignment score. Validate any retuned parameterization by reproducing the paper example S1=abca versus S2=aba, which must yield the alignment triplets F=mmd1m, U1=1234, U2=1203.

## Related tools

- **EpiAlign** (Provides the dynamic-programming local alignment whose mismatch, N-gap, and deletion score functions (NF, DF with epsilon_N/epsilon_D) this skill configures and validates) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Produces the single-track chromatin state label sequences whose state frequencies feed the frequency-based weights W(k,S) used in penalty scaling)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- Paper example reproduced: aligning S1=abca against S2=aba with the chosen penalties must yield the alignment triplets F=mmd1m, U1=1234, U2=1203.
- Score arithmetic check: match pairs contribute W(k,S1)+W(l,S2); mismatches/N-gaps contribute -epsilon_N*(W(k,S1)+W(l,S2)); deletions contribute -epsilon_D*W(k,S); with defaults epsilon_N=1.5 and epsilon_D=1.
- Non-negativity invariant: because 0 is included in the DP max, no alignment score (including the reported M[n1,n2]) is negative for any penalty setting.
- Weight-scheme contrast: switching from equal weights (all W=1) to frequency-based weights must change mismatch/deletion magnitudes, with less common states incurring larger penalties.
- Output artifacts match the expected schema: scoring consumes compressed_sequences.txt and emits alignment_scores.csv.

## Limitations

- Frequency-based weights are only specified in Supplementary section 1 (not included in the provided content), and the paper documents no sensitivity analysis or alternative epsilon values beyond the defaults epsilon_N=1.5 and epsilon_D=1.
- How to construct more specialized substitution matrices for chromatin states is explicitly an important future research question, so tuning is limited to the mismatch/deletion score functions.
- Penalty behavior depends on the upstream chromatin state annotation and compression step; long stretches of a single uncompressed state make the weights, and hence the penalties, uninformative.

## Evidence

- [abstract] EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states.: "EpiAlign is a dynamic programming algorithm that novelly incorporates varying lengths and frequencies of chromatin states."
- [methods] We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence.: "We design the mismatch and deletion score functions based on the weight of each chromatin state in each sequence."
- [methods] equal weights mean that all states are treated equally with the same weight 1 in the sequence S: "equal weights mean that all states are treated equally with the same weight 1 in the sequence S"
- [methods] (ii) frequency-based weights assign larger weights to less common chromatin states (see Supplementary section 1 for details): "(ii) frequency-based weights assign larger weights to less common chromatin states (see Supplementary section 1 for details)"
- [other] Implement the scoring functions: MF(k,l,S1,S2) = W(k,S1)+W(l,S2), NF = -epsilon_N * (W(k,S1)+W(l,S2)), and DF(k,S) = -epsilon_D * W(k,S), with epsilon_N=1.5 and epsilon_D=1 as defaults.: [evidence span withheld — anchor status: not_found]
- [other] the aligner must reproduce the example alignment triplets F=mmd1m, U1=1234, U2=1203 for S1=abca against S2=aba: [evidence span withheld — anchor status: not_found]
- [discussion] how to construct more specialized substitution matrices for chromatin states is an important future research question: "how to construct more specialized substitution matrices for chromatin states is an important future research question"
