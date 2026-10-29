---
name: wig-format-file-writing
description: Use when a ChIP-Seq workflow needs per-bin read-coverage tracks (e.g.
  25 bp bins with end-down-weighted HDF counts) written as WIG files for UCSC genome
  browser visualization or downstream inspection.
when_to_use_negative:
- Input is pair-end data that should be handled as complete fragments via HPeak's
  -pe mode rather than directional single-end extension
- The goal is peak calling/statistical enrichment testing itself rather than producing
  a visualization track
- The alignment is not yet mapped to the genome (WIG bins require chromosomal coordinates)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0092
tools:
- name: HPeak
  role: Generates the WIG-format coverage/visualization file from HDF coverage profiles
    as one of its output types
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Implementation language of the HPeak software package that emits the WIG file
- name: C++
  role: Implementation language of the HPeak software package that emits the WIG file
- name: UCSC genome browser
  role: Downstream consumer of the WIG file for visualizing ChIP-Seq peaks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:e3ce5f8f7f4c009659a96537d8d75e90ade6df3e5e11da1037ca14c6965b0149
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  - UCSC genome browser
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# wig-format-file-writing

## Summary

Generate WIG (wiggle)-format genome-browser track files from weighted hypothetical DNA fragment (HDF) coverage profiles, as HPeak does for visualizing ChIP-Seq read-enriched regions in the UCSC genome browser.

## When to use

Use when a ChIP-Seq workflow needs per-bin read-coverage tracks (e.g. 25 bp bins with end-down-weighted HDF counts) written as WIG files for UCSC genome browser visualization or downstream inspection.

## When NOT to use

- Input is pair-end data that should be handled as complete fragments via HPeak's -pe mode rather than directional single-end extension
- The goal is peak calling/statistical enrichment testing itself rather than producing a visualization track
- The alignment is not yet mapped to the genome (WIG bins require chromosomal coordinates)

## Inputs

- Mapped short-read alignment file (BED or ELAND format) with chromosome, start position, and strand per read
- Read length (e.g. -r 36 for single-end; -isize for pair-end insert size)
- Fragment size range used for extension (e.g. -fmin 100 -fmax 300)

## Outputs

- WIG-format coverage track file (e.g. hdf_coverage.wig) with fixed 25 bp bins and weights in [0, 1]
- Related HPeak outputs produced alongside: peak annotation file and enriched-sequence FASTA file

## How to apply

Extend each mapped short read directionally from its start position by the selected fragment length to form hypothetical DNA fragments (HDFs), following the Robertson et al. extension scheme used by HPeak. Partition the genome into fixed-length 25 bp bins and count HDF coverage per bin, applying the unbalanced end-weighting scheme: full weight (1) across the certain portion of each HDF and a linear ramp from 1 to 0 across the uncertain end region (e.g. 175 bp full coverage plus a 50 bp decay for 175-225 bp size-selected fragments). Serialize the weighted per-bin coverage as a WIG-format file (e.g. hdf_coverage.wig) using fixed 25 bp step records per chromosome. HPeak generates this WIG output automatically via the -wig flag alongside its other outputs (annotation and sequence files).

## Related tools

- **HPeak** (Generates the WIG-format coverage/visualization file from HDF coverage profiles as one of its output types) — github.com/cauyrd/HPeak
- **Perl** (Implementation language of the HPeak software package that emits the WIG file)
- **C++** (Implementation language of the HPeak software package that emits the WIG file)
- **UCSC genome browser** (Downstream consumer of the WIG file for visualizing ChIP-Seq peaks)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Output file hdf_coverage.wig exists and parses as valid WIG with fixed 25 bp bin steps
- All down-weighting values in the uncertain-end ramp region lie within [0, 1]
- WIG track loads correctly in the UCSC genome browser and aligns with the chromosome coordinates in the input BED/ELAND file
- Bins in the certain portion of each HDF carry full weight of 1, matching the size-selection range used (e.g. 175 bp full coverage for 175-225 bp fragments)

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, which limits how precisely the coverage track pinpoints the peak summit
- End weights are tied to the assumed fragment size range; mis-specified -fmin/-fmax or read length (-r) will distort the ramp region
- Genome partitioning bin size is fixed (25 bp in this study), so resolution of the WIG track cannot be finer than the bin size

## Evidence

- [discussion] WIG output as HPeak product: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [discussion] Down-weighting rationale: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] Binning scheme: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [intro] Unbalanced weighting finding: "By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments."
- [readme] WIG flag and read length parameter from README: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
- [discussion] Browser visualization: "easily enabling the visualization of all of the peaks within the UCSC genome browser"
