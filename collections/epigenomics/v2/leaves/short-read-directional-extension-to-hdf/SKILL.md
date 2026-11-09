---
name: short-read-directional-extension-to-hdf
description: Use when ChIP-Seq short-read alignments (BED/ELAND) must be converted
  into a per-bin coverage profile that reflects fragment-length uncertainty, e.g.
  before HMM-based peak calling.
when_to_use_negative:
- When reads are pair-ended with known insert size — HPeak treats the two ends together
  as ONE fragment, so directional extension/weighting of single reads is unnecessary
- When the input is already a binned coverage track or peak calls — re-extending reads
  would double-count coverage
- When strand orientation within a peak must be distinguished to pinpoint the peak
  summit — the current HPeak algorithm does not distinguish reads of different orientation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: HPeak
  role: Implements the directional read extension to HDFs, 25 bp binning, and unbalanced
    end-weighting of the coverage profile
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Driver language for the HPeak pipeline that parses alignments and orchestrates
    HDF construction
- name: C++
  role: Implementation language for the compute-intensive HDF coverage and HMM components
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:68246eec0098e709d1ccffa42e1a51b08714f2eb363108b431e403895512ba4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# short-read-directional-extension-to-hdf

## Summary

Reconstructs a genome-wide HDF (hypothetical DNA fragment) coverage profile from mapped ChIP-Seq short reads by extending each read directionally from its start position and applying HPeak's unbalanced end-weighting scheme to account for uncertainty in actual DNA fragment lengths.

## When to use

Use when ChIP-Seq short-read alignments (BED/ELAND) must be converted into a per-bin coverage profile that reflects fragment-length uncertainty, e.g. before HMM-based peak calling.

## When NOT to use

- When reads are pair-ended with known insert size — HPeak treats the two ends together as ONE fragment, so directional extension/weighting of single reads is unnecessary
- When the input is already a binned coverage track or peak calls — re-extending reads would double-count coverage
- When strand orientation within a peak must be distinguished to pinpoint the peak summit — the current HPeak algorithm does not distinguish reads of different orientation

## Inputs

- Mapped short-read alignment file (BED or ELAND format) with chromosome, start position, and strand per read
- Read length (e.g. -r 36)
- Fragment length selection range (e.g. -fmin 100, -fmax 300) or pair-end insert size (-isize)
- Optional matching control alignment file

## Outputs

- Weighted HDF coverage profile in WIG format (hdf_coverage.wig) with 25 bp bin steps
- Per-bin HDF counts feeding the two-state HMM (GP/ZIP emissions) classification
- Genome browser-ready coverage track and downstream peak annotation inputs

## How to apply

Load the mapped alignment file to extract each read's chromosome, start position, and strand, then extend every read directionally from its start position by the selected fragment length to form hypothetical DNA fragments (HDFs), following the Robertson et al. extension scheme used by HPeak. Partition the genome into fixed-length 25 bp bins and count the HDFs falling in each bin. Apply the unbalanced weighting scheme: assign full weight (1) to bins covered by the certain portion of each HDF and linearly decay the weight from 1 to 0 across the uncertain end region (e.g. 175 bp full coverage plus a 50 bp ramp for 175-225 bp size-selected fragments), because fragment ends are uncertain in single-end data. Write the weighted per-bin profile to a WIG-format file for visualization and downstream two-state HMM analysis. Pair-end runs should instead supply both ends together as ONE fragment via the -pe/-isize options so the insert size is known rather than imputed.

## Related tools

- **HPeak** (Implements the directional read extension to HDFs, 25 bp binning, and unbalanced end-weighting of the coverage profile) — github.com/cauyrd/HPeak
- **Perl** (Driver language for the HPeak pipeline that parses alignments and orchestrates HDF construction)
- **C++** (Implementation language for the compute-intensive HDF coverage and HMM components)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- hdf_coverage.wig exists and is parseable WIG with 25 bp bin steps
- All weights in the end-ramp region lie in [0, 1], decaying from 1 to 0 across the uncertain end portion
- Bins fully inside the certain portion of an HDF carry full weight (1), and no bin exceeds the number of overlapping HDFs
- Total binned coverage is consistent with the supplied read length (-r) and fragment range (-fmin/-fmax) or insert size (-isize)

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, information shown to be informative for pinpointing the peak summit
- In single-end mode the actual fragment length is uncertain, so end-region coverage is only approximately weighted (1-to-0 ramp), not exactly known
- Performance evaluation on experimental ChIP-Seq data should be interpreted with caution because little true positive/negative locus information is available

## Evidence

- [intro] Unbalanced end-weighting rationale: "By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments."
- [discussion] Directional extension to HDF: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] End down-weighting of HDF coverage: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] Fixed 25 bp binning: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [readme] Pair-end insert size handling: "-isize refers to insert size (total length of a pair)"
