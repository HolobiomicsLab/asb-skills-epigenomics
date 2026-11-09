---
name: generalized-poisson-emission-density-estimation
description: Use when modeling ChIP-Seq HDF per-bin count data whose enriched-state
  distribution is overdispersed relative to Poisson and whose background contains
  excess zeros, requiring GP/ZIP emissions for HMM-based bin classification.
when_to_use_negative:
- The input is not directionally extended short-read coverage (HDF counts per fixed
  bin), e.g. it is already a peak list or feature table
- Comparative histone modification data better suited to ChIPDiff, which is restricted
  to that analysis per the article
- A quantification of false detection rate is the primary goal, since the HMM-based
  method is not well-suited for quantifying FDR
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Implements the two-state HMM with GP (enriched) and ZIP (background) emissions
    on 25 bp HDF coverage bins, with Viterbi parameter estimation
  repo: https://github.com/cauyrd/HPeak
- name: Perl
  role: Driver layer of the HPeak software package that wraps the HMM computation
- name: C++
  role: Performance layer implementing the GP/ZIP emission and Viterbi HMM computation
    in the HPeak package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:da57dd29d74eec9462cb1de595925ce97517ec3c7e97a589a0f2ab33f38d5a2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cauyrd/HPeak
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# generalized-poisson-emission-density-estimation

## Summary

Fit Generalized Poisson (GP) and zero-inflated Poisson (ZIP) emission densities to per-bin HDF coverage counts so a two-state HMM can classify 25 bp bins as ChIP-enriched or background in HPeak-style ChIP-Seq peak calling.

## When to use

Use when modeling ChIP-Seq HDF per-bin count data whose enriched-state distribution is overdispersed relative to Poisson and whose background contains excess zeros, requiring GP/ZIP emissions for HMM-based bin classification.

## When NOT to use

- The input is not directionally extended short-read coverage (HDF counts per fixed bin), e.g. it is already a peak list or feature table
- Comparative histone modification data better suited to ChIPDiff, which is restricted to that analysis per the article
- A quantification of false detection rate is the primary goal, since the HMM-based method is not well-suited for quantifying FDR

## Inputs

- Aligned ChIP-Seq short reads (BED or ELAND format; single-end with -r read length, or pair-end with -isize insert size)
- Optional aligned control (input) sample reads for bivariate GP/ZIP classification
- Genome partition into fixed-length 25 bp bins with HDF coverage counts

## Outputs

- Fitted GP emission parameters for the ChIP-enriched state and ZIP parameters for the background state
- Per-bin state assignments (enriched vs background) for every 25 bp bin (e.g. hpeak_hmm_states.txt)
- Merged blocks of consecutive ChIP-enriched bins with summit location and p-value, plus WIG, FASTA sequence, and annotation files

## How to apply

Extend each short read directionally into hypothetical DNA fragments (HDFs) following Robertson et al., partition the genome into fixed-length 25 bp bins, and count HDFs per bin while applying the unbalanced weighting scheme that gradually down-weights coverage from the end portion of each HDF. Obtain quick initial HMM parameter estimates from summary statistics of the per-bin counts, then fit the two-state HMM with the Viterbi algorithm, using the GP density for the ChIP-enriched state — because GP was shown to fit enriched HDF counts much better than Poisson and slightly better than negative binomial — and the ZIP density for the background state. For paired treated/control experiments, extend the emissions to the bivariate GP/ZIP variant so bins are classified jointly from both channels. Run Viterbi decoding to assign each 25 bp bin a state and merge consecutive enriched bins into ChIP-enriched blocks; the compute-heavy emission/decoding code is implemented in C++ within HPeak's Perl/C++ package.

## Related tools

- **HPeak** (Implements the two-state HMM with GP (enriched) and ZIP (background) emissions on 25 bp HDF coverage bins, with Viterbi parameter estimation) — https://github.com/cauyrd/HPeak
- **Perl** (Driver layer of the HPeak software package that wraps the HMM computation)
- **C++** (Performance layer implementing the GP/ZIP emission and Viterbi HMM computation in the HPeak package)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- The GP density fits the HDF counts of ChIP-enriched regions much better than Poisson and slightly better than negative binomial (distribution-fit comparison)
- The ZIP density fits background HDF counts slightly better than both Poisson and NB distributions
- Every 25 bp bin of the genome partition receives exactly one state label, and consecutive enriched bins merge into contiguous ChIP-enriched blocks
- Identified peaks show higher prevalence of the expected transcription factor binding motifs in enriched vs control sequences than competing peak callers (e.g. best true positive rate on NRSF ChIP-Seq data)
- For paired data, the bivariate GP/ZIP emissions consume both treatment and control channels, and enrichment summaries are consistent between them

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, although such information is informative for pinpointing peak summits
- The ZIP fit result for background regions was reported as 'data not shown', and whether NB or other distributions might fit better remains incompletely documented
- Little information on true positive/negative loci exists for experimental ChIP-Seq data, so performance comparisons should be interpreted with caution and the method is not well-suited for quantifying false detection rate

## Evidence

- [discussion] GP emission for enriched state fit: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] ZIP emission for background fit: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [discussion] Two-state HMM over HDF coverage profile: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] Viterbi estimation and bin size: "For HMM parameter estimation, we use the well-established Viterbi algorithm [50]. In the initial step, we use summary statistics to get a quick and rough estimate"
- [discussion] Down-weighting scheme rationale: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] Implementation language: "We have implemented HPeak in a software package using Perl and C++."
- [readme] README invocation and parameters: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36"
