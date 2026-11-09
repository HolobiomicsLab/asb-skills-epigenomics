---
name: zero-inflated-poisson-background-modeling
description: Use when calling ChIP-enriched regions from HDF per-bin counts (e.g.,
  25 bp bins) and you need a discrete emission model that explicitly handles excess
  zeros in background coverage alongside an overdispersed enriched-state signal.
when_to_use_negative:
- Input is already a set of called peak intervals or a feature table rather than raw
  aligned short reads needing coverage-based classification
- Comparative histone modification datasets of the type handled by ChIPDiff, which
  is restricted to comparative histone data rather than general peak calling
- You need to distinguish reads of different orientation within a peak to pinpoint
  summits — the current HPeak algorithm does not support this
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: HPeak
  role: Two-state HMM peak caller whose background state uses the ZIP emission and
    enriched state uses the GP emission, with Viterbi-based parameter estimation
  repo: github.com/cauyrd/HPeak
- name: C++
  role: Performance-critical implementation of the HMM with GP/ZIP emissions
- name: Perl
  role: Wrapper/driver layer of the HPeak software package invoking the C++ HMM routines
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:650cd0302f970ddcebcfb6443c0ee5c1270783a26e5caa52ea84d498f7b3e100
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# zero-inflated-poisson-background-modeling

## Summary

Model background HDF coverage in ChIP-Seq bins with a zero-inflated Poisson (ZIP) emission within HPeak's two-state HMM, since background bins are dominated by zeros that plain Poisson underfits. The ZIP is paired with a Generalized Poisson emission for the ChIP-enriched state during Viterbi-based parameter estimation.

## When to use

Use when calling ChIP-enriched regions from HDF per-bin counts (e.g., 25 bp bins) and you need a discrete emission model that explicitly handles excess zeros in background coverage alongside an overdispersed enriched-state signal.

## When NOT to use

- Input is already a set of called peak intervals or a feature table rather than raw aligned short reads needing coverage-based classification
- Comparative histone modification datasets of the type handled by ChIPDiff, which is restricted to comparative histone data rather than general peak calling
- You need to distinguish reads of different orientation within a peak to pinpoint summits — the current HPeak algorithm does not support this

## Inputs

- Per-bin HDF coverage counts on a fixed 25 bp genome-wide partition
- Aligned ChIP-Seq reads (BED or ELAND format, single-end -t or paired-end -pe) optionally with matched control reads (-c)
- Read length (-r) and, for paired-end data, insert size (-isize)

## Outputs

- Per-bin state assignments (enriched vs. background) in hpeak_hmm_states.txt
- Merged ChIP-enriched blocks (peaks) with summit location and p-value
- Optional WIG visualization file, FASTA sequence file for motif analysis, and annotation file

## How to apply

After partitioning the genome into fixed-length 25 bp bins and counting HDFs per bin (with the unbalanced down-weighting at HDF ends), compute rough initial HMM parameters from summary statistics of these counts. Then run the two-state HMM implemented in C++ (mirroring HPeak's Perl/C++ package), assigning a Generalized Poisson density to the ChIP-enriched state and a zero-inflated Poisson density to the background state, estimating parameters with the Viterbi algorithm. For paired treated/control data, extend to the bivariate GP/ZIP variant so bins are classified jointly from both channels. Finally, Viterbi decoding assigns each bin to enriched or background, and consecutive enriched bins merge into ChIP-enriched blocks written to hpeak_hmm_states.txt.

## Related tools

- **HPeak** (Two-state HMM peak caller whose background state uses the ZIP emission and enriched state uses the GP emission, with Viterbi-based parameter estimation) — github.com/cauyrd/HPeak
- **C++** (Performance-critical implementation of the HMM with GP/ZIP emissions)
- **Perl** (Wrapper/driver layer of the HPeak software package invoking the C++ HMM routines)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Output file hpeak_hmm_states.txt exists and contains a state label for every 25 bp bin of the input genome partition
- Distributional fit check: the ZIP distribution produces a slightly better fit than both Poisson and NB distributions for background bins, and the GP distribution fits enriched-state HDF counts much better than Poisson
- Merged consecutive enriched bins form contiguous ChIP-enriched blocks consistent with reference HPeak two-state HMM behavior
- Identified peaks show higher prevalence of the expected transcription factor binding motifs in ChIP-enriched sequences relative to control sequences (motif enrichment check, e.g., via MatInspector with Genomatix MatBase 7.0 matrices)
- On benchmark data such as NRSF ChIP-Seq, the true positive rate should be at or near the best among compared peak callers

## Limitations

- There is little information on true positive and true negative loci in experimental ChIP-Seq data, making evaluation criteria for peak callers hard to define; performance results should be interpreted with caution
- The method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods
- The ZIP fit result for background regions was reported only as 'data not shown', so background distributional-fit evidence is incomplete
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, which limits summit pinpointing

## Evidence

- [discussion] ZIP distribution fits background HDF counts: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [discussion] Two-state HMM classification with Viterbi: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] GP better fits enriched-state counts: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] Implementation language of HPeak: "We have implemented HPeak in a software package using Perl and C++."
- [readme] HMM peak-finding purpose per README: "HPeak is a hidden Markov model-based approach that can accurately pinpoint regions to where significantly more sequence reads map."
