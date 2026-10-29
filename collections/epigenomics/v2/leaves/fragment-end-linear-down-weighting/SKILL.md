---
name: fragment-end-linear-down-weighting
description: Use when reconstructing coverage profiles from directionally extended
  short ChIP-Seq reads and fragment sizes are only known within a size-selection range
  (e.g. 175-225 bp), so HDF ends are uncertain.
when_to_use_negative:
- Paired-end data where the true fragment ends are directly observed — HPeak treats
  both ends together as ONE fragment, so end inference/weighting is unnecessary
- Analyses requiring strand-aware peak-summit refinement — the current HPeak algorithm
  does not distinguish reads of different orientation within a peak
- Genome binnings at resolutions other than the fixed bin scheme expected by the HMM
  (25 bp in this study) without re-validating the weighted profile
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Implements directional read extension to HDFs, 25 bp binning, and the unbalanced
    end down-weighting scheme that produces the weighted coverage profile
  repo: http://www.sph.umich.edu/csg/qin/HPeak
- name: Perl
  role: Language of the HPeak package driver (HPeak.pl) invoked to run the weighting/peak-calling
    pipeline
- name: C++
  role: Implementation language of the HPeak software package alongside Perl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:4b8cf2aba78bb6e089527909a2725ef3272785b7da00b25564dda465c168b5a0
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fragment-end-linear-down-weighting

## Summary

HPeak's scheme for reconstructing genome-wide ChIP-Seq coverage profiles by extending each short read into a hypothetical DNA fragment (HDF) and linearly down-weighting coverage in the bins covered by the HDF's uncertain end portion, so fragment-length uncertainty does not inflate signal at fragment ends.

## When to use

Use when reconstructing coverage profiles from directionally extended short ChIP-Seq reads and fragment sizes are only known within a size-selection range (e.g. 175-225 bp), so HDF ends are uncertain.

## When NOT to use

- Paired-end data where the true fragment ends are directly observed — HPeak treats both ends together as ONE fragment, so end inference/weighting is unnecessary
- Analyses requiring strand-aware peak-summit refinement — the current HPeak algorithm does not distinguish reads of different orientation within a peak
- Genome binnings at resolutions other than the fixed bin scheme expected by the HMM (25 bp in this study) without re-validating the weighted profile

## Inputs

- Mapped short-read alignment file in BED (or ELAND) format with chromosome, start position, and strand per read
- Read length (e.g. -r 36 for single-end; -r 100 for paired-end)
- Size-selection range or insert size defining the uncertain end region (-fmin/-fmax, e.g. 100-300; or -isize 200 for paired-end)

## Outputs

- Weighted HDF coverage profile in WIG format (e.g. hdf_coverage.wig) with 25 bp bins
- Binned coverage profile feeding the two-state HMM classification of bins as ChIP-enriched or background

## How to apply

Parse the mapped alignment (BED/ELAND) to get each read's chromosome, start, and strand, then extend every read directionally from its start position following the Robertson et al. scheme to form HDFs. Partition the genome into fixed-length 25 bp bins and count HDFs per bin. Assign full weight (1) to bins covered by the certain portion of each HDF and linearly decay the weight from 1 to 0 across the uncertain end region (e.g. 175 bp full coverage plus a 50 bp ramp for 175-225 bp size-selected fragments; HPeak parameters such as -fmin/-fmax or -isize define this range). Write the weighted per-bin coverage to a WIG-format file, which then feeds HPeak's two-state HMM with GP/ZIP emissions. The rationale is that reads only mark the start of a fragment, so coverage contribution should fade where the true fragment end is unknown.

## Related tools

- **HPeak** (Implements directional read extension to HDFs, 25 bp binning, and the unbalanced end down-weighting scheme that produces the weighted coverage profile) — http://www.sph.umich.edu/csg/qin/HPeak
- **Perl** (Language of the HPeak package driver (HPeak.pl) invoked to run the weighting/peak-calling pipeline)
- **C++** (Implementation language of the HPeak software package alongside Perl)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -wig
```

## Evaluation signals

- hdf_coverage.wig exists and parses as valid WIG with 25 bp bin steps
- All weights in the linear ramp region lie in [0, 1], decaying monotonically from 1 to 0
- Bins in the certain portion of each HDF carry full weight (1), matching the e.g. 175 bp full-coverage / 50 bp ramp split for 175-225 bp fragments
- Weighted per-bin counts match the sum of per-HDF weighted contributions for a sampled locus
- Resulting peak calls show higher motif enrichment (e.g. MatInspector scan against Genomatix MatBase 7.0) without loss of sensitivity relative to unweighted coverage

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such information helps pinpoint peak summits
- Performance evaluation on experimental data is hard to benchmark since little true positive/negative locus information is available
- The weighting depends on the assumed size-selection range; if -fmin/-fmax or -isize poorly reflect actual fragment sizes, the ramp misplaces coverage

## Evidence

- [intro] By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments.: "By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments."
- [discussion] Directional extension to HDFs: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] End down-weighting: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] 25 bp binning: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] WIG output: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] Pair-end handling: "Fully supports pair-end data by treating two ends together as ONE fragment"
- [readme] Size range parameters: "-t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
