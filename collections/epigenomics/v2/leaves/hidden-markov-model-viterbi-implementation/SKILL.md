---
name: hidden-markov-model-viterbi-implementation
description: Use when short-read ChIP-Seq coverage has been binned (e.g., 25 bp HDF
  counts) and consecutive bins must be statistically classified into enriched vs.
  background states with a probability model that explicitly captures overdispersion
  and excess zeros.
when_to_use_negative:
- Input is a comparative histone modification dataset better served by ChIPDiff, which
  HPeak's authors note is 'restricted to analyzing comparative histone modification
  data'
- Reads of different orientation within a peak must be distinguished to pinpoint peak
  summits — the current HPeak algorithm does not use orientation information
- The analysis requires a well-calibrated false detection rate; the paper notes the
  method 'is not well-suited for quantifying false detection rate'
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: HPeak
  role: Reference two-state HMM peak-caller whose GP/ZIP emission model and Viterbi
    parameter estimation this skill reimplements
  repo: github.com/cauyrd/HPeak
- name: C++
  role: Language for the performance-critical HMM/Viterbi implementation, mirroring
    HPeak's Perl/C++ package
- name: Perl
  role: Driver/wrapper layer of the HPeak package invoking the C++ HMM engine
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:f6d47e9b9e07bff7db42f0651c1563a449f07d5762f0a9d3c73d2e4d4cd67d08
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hidden-markov-model-viterbi-implementation

## Summary

Implement a two-state hidden Markov model (as in HPeak) over fixed-length genomic bin coverage, with a Generalized Poisson emission for the ChIP-enriched state and a zero-inflated Poisson emission for the background state, using Viterbi-based parameter estimation to classify each 25 bp bin as enriched or background.

## When to use

Use when short-read ChIP-Seq coverage has been binned (e.g., 25 bp HDF counts) and consecutive bins must be statistically classified into enriched vs. background states with a probability model that explicitly captures overdispersion and excess zeros.

## When NOT to use

- Input is a comparative histone modification dataset better served by ChIPDiff, which HPeak's authors note is 'restricted to analyzing comparative histone modification data'
- Reads of different orientation within a peak must be distinguished to pinpoint peak summits — the current HPeak algorithm does not use orientation information
- The analysis requires a well-calibrated false detection rate; the paper notes the method 'is not well-suited for quantifying false detection rate'

## Inputs

- Aligned ChIP-Seq short reads (BED or ELAND format; SAM/BAM planned) with known read length (-r)
- Optional aligned control sample reads for paired treated/control analysis
- Genome partition into fixed-length 25 bp bins with per-bin HDF coverage counts

## Outputs

- Per-bin state assignments (ChIP-enriched vs. background) for every 25 bp bin, e.g., hpeak_hmm_states.txt
- Merged blocks of consecutive ChIP-enriched bins (peaks) with summit location and p-value
- Downstream files: WIG coverage track, FASTA peak sequences, and peak annotation file (-wig -seq -ann)

## How to apply

Partition the genome into fixed-length 25 bp bins and count directionally extended hypothetical DNA fragments (HDFs, built per Robertson et al. by extending each short read) falling in each bin, applying HPeak's unbalanced weighting scheme that gradually down-weights coverage at the end portions of each HDF to account for fragment-length uncertainty. Compute initial rough HMM parameter estimates from summary statistics of the per-bin counts, then estimate parameters with the Viterbi algorithm. Emit a Generalized Poisson density for the ChIP-enriched state (GP fits HDF counts there better than Poisson and slightly better than negative binomial) and a zero-inflated Poisson for background (ZIP fits slightly better than Poisson and NB). For paired treated/control samples, extend to the bivariate GP/ZIP variant so bins are classified jointly from both channels. Run Viterbi decoding to assign state labels, merge consecutive enriched bins into blocks, and write per-bin states and merged blocks to an output file such as hpeak_hmm_states.txt. Implement the hot loop in C++ for performance, as HPeak does with its Perl/C++ package.

## Related tools

- **HPeak** (Reference two-state HMM peak-caller whose GP/ZIP emission model and Viterbi parameter estimation this skill reimplements) — github.com/cauyrd/HPeak
- **C++** (Language for the performance-critical HMM/Viterbi implementation, mirroring HPeak's Perl/C++ package)
- **Perl** (Driver/wrapper layer of the HPeak package invoking the C++ HMM engine)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Output state file exists and contains exactly one state label for every 25 bp bin of the input genome partition
- Enriched-state emission residuals are consistent with a Generalized Poisson fit; background with a zero-inflated Poisson (GP 'much better fit than the Poisson distribution' for enriched counts; ZIP 'slightly better fit than both Poisson and NB' for background)
- Merged enriched blocks consist of consecutive bins all labeled enriched by Viterbi decoding
- For paired treated/control runs, the bivariate GP/ZIP HMM produces classifications consistent with joint two-channel behavior (treatment enrichment against control background)
- Behavior matches reference HPeak output on benchmark data such as the NRSF ChIP-Seq set, where HPeak 'showed the best true positive rate'

## Limitations

- Background-region ZIP fit superiority was reported as 'data not shown', so the emission-choice evidence for background is not fully documented
- Current algorithm does not distinguish reads of different orientation within a peak, which limits summit pinpointing
- Performance evaluation should be interpreted with caution because true positive/negative loci are largely unknown for experimental ChIP-Seq data

## Evidence

- [discussion] two-state HMM with GP/ZIP emissions: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] Viterbi parameter estimation: "For HMM parameter estimation, we use the well-established Viterbi algorithm [50]. In the initial step, we use summary statistics to get a quick and rough estimate"
- [discussion] GP emission choice for enriched state: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] unbalanced weighting rationale: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [readme] HMM-based peak calling overall: "HPeak is a hidden Markov model-based approach that can accurately pinpoint regions to where significantly more sequence reads map."
