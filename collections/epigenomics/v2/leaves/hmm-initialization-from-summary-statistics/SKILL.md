---
name: hmm-initialization-from-summary-statistics
description: Use when calling ChIP-Seq peaks with a two-state HMM over 25 bp HDF coverage
  bins and you need defensible starting values for transition and GP/ZIP emission
  parameters prior to Viterbi refinement.
when_to_use_negative:
- You need to quantify a false detection rate for called peaks — the paper states
  the method 'is not well-suited for quantifying false detection rate', so evaluation-focused
  pipelines should not rely on it for FDR
- You need peak summits distinguished by read orientation — HPeak 'does not distinguish
  reads of different orientation within a peak', a feature deferred to a future release
- You need a refined distributional fit comparison (e.g., Poisson vs NB vs GP/ZIP)
  rather than peak classification — the paper's distribution-fitting analyses are
  descriptive support, not the classification task itself
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: HPeak
  role: Implements the two-state HMM over HDF coverage profiles; its initialization
    from summary statistics seeds Viterbi-based parameter estimation with GP (enriched)
    and ZIP (background) emissions
  repo: github.com/cauyrd/HPeak
- name: C++
  role: Performance-critical implementation of the two-state HMM parameter estimation
    and Viterbi decoding in the HPeak software package
- name: Perl
  role: Driver layer of the HPeak package orchestrating input parsing and pipeline
    execution around the C++ HMM routines
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:6bf3f1b9af925199b8ecd7c1a0e965bc2fd9f40e42e2c890d568732c5819a9a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# hmm-initialization-from-summary-statistics

## Summary

Initialize HPeak's two-state hidden Markov model (ChIP-enriched state with Generalized Poisson emissions, background state with zero-inflated Poisson emissions) using quick, rough parameter estimates computed from summary statistics of per-bin HDF coverage counts, before refining them with Viterbi-based estimation.

## When to use

Use when calling ChIP-Seq peaks with a two-state HMM over 25 bp HDF coverage bins and you need defensible starting values for transition and GP/ZIP emission parameters prior to Viterbi refinement.

## When NOT to use

- You need to quantify a false detection rate for called peaks — the paper states the method 'is not well-suited for quantifying false detection rate', so evaluation-focused pipelines should not rely on it for FDR
- You need peak summits distinguished by read orientation — HPeak 'does not distinguish reads of different orientation within a peak', a feature deferred to a future release
- You need a refined distributional fit comparison (e.g., Poisson vs NB vs GP/ZIP) rather than peak classification — the paper's distribution-fitting analyses are descriptive support, not the classification task itself

## Inputs

- Aligned ChIP-Seq short reads in BED or ELAND format (treatment channel; optional matched control channel)
- Read length specification (-r, e.g., 36 for single-end; insert size via -isize 200 for pair-end)
- Fragment length extension range (-fmin 100 -fmax 300 for single-end) used to build HDFs
- Genome partition into fixed-length 25 bp bins with per-bin HDF count profile

## Outputs

- Initial rough estimates of two-state HMM parameters (state priors, transition probabilities, GP emission for enriched state, ZIP emission for background state)
- Viterbi-refined HMM parameters after convergence
- Per-bin state assignments (hpeak_hmm_states.txt) labeling every 25 bp bin as ChIP-enriched or background
- Merged blocks of consecutive ChIP-enriched bins

## How to apply

First form hypothetical DNA fragments (HDFs) by directionally extending each short read (Robertson et al. approach) and partition the genome into fixed-length 25 bp bins, counting HDFs per bin with HPeak's unbalanced weighting scheme that down-weights coverage at fragment ends. Compute summary statistics (e.g., mean and variance of per-bin HDF counts) to obtain a quick and rough initial estimate of the two-state HMM parameters: state priors, transition probabilities, the GP emission for the ChIP-enriched state, and the ZIP emission for the background state. These rough estimates seed the Viterbi-based parameter estimation, which is the well-established HPeak refinement step. Rationale for GP/ZIP: the paper's fit comparisons show GP fits enriched-region HDF counts much better than Poisson and slightly better than NB, while ZIP fits background counts slightly better than Poisson and NB. For paired treated/control data, extend to the bivariate GP/ZIP emission variant so the HMM classifies bins jointly from both channels.

## Related tools

- **HPeak** (Implements the two-state HMM over HDF coverage profiles; its initialization from summary statistics seeds Viterbi-based parameter estimation with GP (enriched) and ZIP (background) emissions) — github.com/cauyrd/HPeak
- **C++** (Performance-critical implementation of the two-state HMM parameter estimation and Viterbi decoding in the HPeak software package)
- **Perl** (Driver layer of the HPeak package orchestrating input parsing and pipeline execution around the C++ HMM routines)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -interfiles
```

## Evaluation signals

- Output contains a state label for every 25 bp bin of the input genome partition — no missing or extra bins
- Classified ChIP-enriched state uses Generalized Poisson emissions and background state uses zero-inflated Poisson emissions, matching the paper's Methods
- Viterbi-refined parameters should yield a better model of the HDF count profile than the initial summary-statistic estimates (initial estimates are explicitly 'quick and rough')
- Consecutive bins assigned to the enriched state merge into contiguous blocks consistent with reference HPeak two-state HMM behavior
- On benchmark data (e.g., NRSF ChIP-Seq), enrichment of expected transcription factor binding motifs under called peaks should not degrade versus other peak callers, per the paper's evaluation

## Limitations

- The initialization is deliberately rough: 'In the initial step, we use summary statistics to get a quick and rough estimate', so downstream Viterbi refinement is essential before decoding
- For background regions, the ZIP fit result over Poisson/NB was reported as 'data not shown', so the background emission choice rests on partially unpublished fit comparisons
- The method is not well-suited for quantifying false detection rate, and performance evaluation results 'should be interpreted with caution' given limited true positive/negative loci
- HPeak does not distinguish reads of different orientation within a peak, limiting summit precision

## Evidence

- [discussion] For HMM parameter estimation, we use the well-established Viterbi algorithm [50]. In the initial step, we use summary statistics to get a quick and rough estimate: "For HMM parameter estimation, we use the well-established Viterbi algorithm [50]. In the initial step, we use summary statistics to get a quick and rough estimate"
- [discussion] HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [readme] HPeak is a hidden Markov model-based approach that can accurately pinpoint regions to where significantly more sequence reads map.: "HPeak is a hidden Markov model-based approach that can accurately pinpoint regions to where significantly more sequence reads map."
