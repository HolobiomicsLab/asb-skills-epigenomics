---
name: bivariate-gp-zip-paired-sample-classification
description: Use when paired treated/control ChIP-Seq aligned reads must be classified
  into ChIP-enriched versus background bins via an HMM whose emissions model both
  channels simultaneously (GP for enriched, ZIP for background).
when_to_use_negative:
- Input is a single-sample ChIP-Seq experiment with no matched control reads — the
  univariate GP/ZIP HMM variant suffices; the bivariate extension requires both channels
- Data are comparative histone-modification tracks from a method like ChIPDiff rather
  than read-level ChIP-Seq alignment data
- The goal is quantifying a false detection rate — the paper notes its method is not
  well-suited for quantifying FDR
edam_operation: http://edamontology.org/operation_0415
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Two-state HMM peak-caller whose paired-data variant uses bivariate GP (enriched)
    and ZIP (background) emissions to jointly classify treatment/control bins
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Part of the HPeak software package implementation (driver/scripting layer)
- name: C++
  role: Performance-critical HMM/Viterbi implementation layer of the HPeak package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:cab3703b16bfa3e948bf2372f7ea1ff89cfd21172cdcd516f804c3be300b19e9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_0415
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# bivariate-gp-zip-paired-sample-classification

## Summary

Extend HPeak's two-state HMM to paired ChIP treatment/control data by classifying fixed-length 25 bp bins jointly from both read channels, using a bivariate Generalized Poisson emission for ChIP-enriched bins and a bivariate zero-inflated Poisson emission for background bins.

## When to use

Use when paired treated/control ChIP-Seq aligned reads must be classified into ChIP-enriched versus background bins via an HMM whose emissions model both channels simultaneously (GP for enriched, ZIP for background).

## When NOT to use

- Input is a single-sample ChIP-Seq experiment with no matched control reads — the univariate GP/ZIP HMM variant suffices; the bivariate extension requires both channels
- Data are comparative histone-modification tracks from a method like ChIPDiff rather than read-level ChIP-Seq alignment data
- The goal is quantifying a false detection rate — the paper notes its method is not well-suited for quantifying FDR

## Inputs

- Aligned ChIP-Seq treatment reads (BED or ELAND format; single-end via -t or pair-end via -pe)
- Aligned control ChIP-Seq reads supplied via -c (pair-end control with -pec)
- Read length (-r) and, for pair-end data, insert size (-isize)
- Species/genome specification (-sp HUMAN/MOUSE)

## Outputs

- Per-bin state assignments for every 25 bp bin of the genome partition (hpeak_hmm_states.txt: enriched vs background labels)
- Merged blocks of consecutive ChIP-enriched bins with summit location and p-value
- Optional WIG coverage file, FASTA sequence of enriched regions, and peak annotation file

## How to apply

Extend each short read directionally into HDFs (Robertson et al. style, read length given via -r, insert size via -isize for pair-end), partition the genome into fixed-length 25 bp bins, and count HDFs per bin while applying HPeak's unbalanced weighting scheme that gradually down-weights HDF coverage in bins covered by the end portion of each HDF. For paired data, feed both treatment and control per-bin count profiles into the two-state HMM, using the bivariate GP density for the enriched state and the bivariate ZIP density for the background state; initialize HMM parameters from summary statistics of the per-bin HDF counts and refine with Viterbi-based estimation. Run Viterbi decoding to label every 25 bp bin as enriched or background, then merge consecutive enriched bins into blocks. The rationale is that GP fits HDF counts in ChIP-enriched regions better than Poisson (and slightly better than NB), while ZIP fits background counts slightly better than Poisson and NB, so joint bivariate emissions let control signal moderate enrichment calls bin-by-bin.

## Related tools

- **HPeak** (Two-state HMM peak-caller whose paired-data variant uses bivariate GP (enriched) and ZIP (background) emissions to jointly classify treatment/control bins) — github.com/cauyrd/HPeak
- **Perl** (Part of the HPeak software package implementation (driver/scripting layer))
- **C++** (Performance-critical HMM/Viterbi implementation layer of the HPeak package)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- hpeak_hmm_states.txt exists and contains a state label for every 25 bp bin of the input genome partition
- Classified enriched blocks are consistent with the reference HPeak behavior: GP emission for the enriched state, ZIP emission for the background state
- With paired data supplied (-t plus -c), peak calls should shift toward regions where treatment counts exceed control counts relative to single-channel calls
- Identified enriched regions should show higher prevalence of the expected transcription factor binding motifs than control sequences (HPeak showed higher motif enrichment than other approaches)
- Output includes summit location and p-value per peak, and WIG output visualizes correctly in the UCSC genome browser

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such information is informative for pinpointing peak summits (planned for a future release)
- Performance evaluation criteria are challenging because little information on true positive and true negative loci is available for experimental ChIP-Seq data; results should be interpreted with caution
- The method is not well-suited for quantifying false detection rate, which may disadvantage some compared methods
- Background-region ZIP fit results versus NB were reported only as 'data not shown', so the background emission fit is less thoroughly documented than the enriched-state GP fit

## Evidence

- [discussion] HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background: "HPeak applies a two-state HMM on the HDF coverage profile to distinguish blocks of consecutive ChIP-enriched bins from the background"
- [discussion] we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions: "the GP distribution shows a much better fit than the Poisson distribution and a slightly better fit than the NB distributions"
- [discussion] the ZIP distribution produces a slightly better fit than both Poisson and NB distributions: "the ZIP distribution produces a slightly better fit than both Poisson and NB distributions"
- [readme] Fully supports pair-end data by treating two ends together as ONE fragment: "Fully supports pair-end data by treating two ends together as ONE fragment"
