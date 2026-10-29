---
name: binned-genome-coverage-construction
description: Use when converting mapped single-end short reads (BED/ELAND) into a
  binned genome-wide coverage profile for peak calling, especially when fragment sizes
  are only approximately known from size selection.
when_to_use_negative:
- The input is already pair-end with confident insert sizes handled natively — HPeak's
  pair-end mode (-pe -isize) treats both ends as one fragment and does not need directional
  extension with uncertain ends
- The goal is histone-modification comparative analysis across conditions — the article
  notes ChIPDiff is the method restricted to comparative histone data, whereas HPeak
  targets TF binding-site enriched regions
- Coverage must distinguish read orientation within peaks — the article states the
  current HPeak algorithm does not distinguish reads of different orientation within
  a peak
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Implements directional HDF extension, 25 bp binning, unbalanced end-weighting,
    and WIG output as the pre-HMM coverage construction stage
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Driver/entry point (HPeak.pl) that parses BED/ELAND alignments and orchestrates
    the coverage construction pipeline with C++ core routines
- name: C++
  role: Implements the computational core of HPeak, including binned HDF counting
    and weighting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:a45478dce0e7a19dcba121ee80401094284c92b4368cdb6d238cbc1f9098b125
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# binned-genome-coverage-construction

## Summary

Reconstructs a genome-wide ChIP-Seq coverage profile by directionally extending each mapped short read into a hypothetical DNA fragment (HDF), counting HDFs per fixed 25 bp bin, and applying HPeak's unbalanced end-weighting so uncertain fragment ends contribute decaying coverage. This accounts for fragment-length uncertainty before HMM-based peak calling.

## When to use

Use when converting mapped single-end short reads (BED/ELAND) into a binned genome-wide coverage profile for peak calling, especially when fragment sizes are only approximately known from size selection.

## When NOT to use

- The input is already pair-end with confident insert sizes handled natively — HPeak's pair-end mode (-pe -isize) treats both ends as one fragment and does not need directional extension with uncertain ends
- The goal is histone-modification comparative analysis across conditions — the article notes ChIPDiff is the method restricted to comparative histone data, whereas HPeak targets TF binding-site enriched regions
- Coverage must distinguish read orientation within peaks — the article states the current HPeak algorithm does not distinguish reads of different orientation within a peak

## Inputs

- Mapped short-read alignment file in BED or ELAND format (chromosome, start, strand per read); treatment and optional control
- Read length (e.g. -r 36) and fragment size selection range (e.g. -fmin 100, -fmax 300), or insert size for pair-end data (-isize 200)
- Genome specification (HUMAN/MOUSE default in HPeak)

## Outputs

- Weighted per-bin HDF coverage profile in WIG format (hdf_coverage.wig) with 25 bp bin steps, ready for HPeak HMM peak calling
- Intermediate HDF count files per bin (with -interfiles)

## How to apply

Parse the mapped alignment file (BED or ELAND) to obtain each read's chromosome, start position, and strand, then extend every read directionally from its start position to form HDFs following the Robertson et al. scheme, using fragment lengths appropriate to the size selection (e.g. -fmin 100 -fmax 300). Partition the genome into fixed-length 25 bp bins and accumulate HDF coverage per bin as in HPeak. Apply the unbalanced weighting scheme: bins covered by the certain portion of each HDF receive full weight (1), while bins in the uncertain end region receive weights decaying from 1 to 0 (e.g. full coverage over a certain 175 bp core with a 50 bp linear ramp for 175–225 bp size-selected fragments). For pair-end data, treat the two ends together as ONE fragment and use the insert size (-isize) instead of directional extension. Emit the weighted per-bin profile as a WIG-format file (hdf_coverage.wig), which HPeak then feeds to its two-state HMM with GP and ZIP emissions.

## Related tools

- **HPeak** (Implements directional HDF extension, 25 bp binning, unbalanced end-weighting, and WIG output as the pre-HMM coverage construction stage) — github.com/cauyrd/HPeak
- **Perl** (Driver/entry point (HPeak.pl) that parses BED/ELAND alignments and orchestrates the coverage construction pipeline with C++ core routines)
- **C++** (Implements the computational core of HPeak, including binned HDF counting and weighting)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -wig -interfiles
```

## Evaluation signals

- hdf_coverage.wig exists and parses as valid WIG with 25 bp fixed bin steps
- All weights assigned in the uncertain end ramp region lie in [0, 1], with full weight 1 in the certain portion of each HDF and monotonic decay to 0
- HDFs are extended directionally from each read's start position per strand, consistent with the Robertson et al. extension scheme
- For pair-end input, each paired-end read is counted as ONE fragment of length -isize rather than two independently extended reads
- Downstream HPeak HMM on the weighted profile returns peak calls with summit locations and p-values, and WIG tracks visualize correctly in the UCSC genome browser

## Limitations

- Fragment lengths are only estimated from size selection; the unbalanced weighting mitigates but does not eliminate uncertainty in actual fragment lengths
- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such information can pinpoint peak summits
- Read length (-r) must be supplied correctly by the user; the README notes this parameter is important

## Evidence

- [discussion] Directional HDF extension: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] Fixed 25 bp binning: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [intro] Unbalanced end-weighting rationale: "By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments."
- [discussion] End down-weighting mechanism: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [discussion] WIG output artifact: "These include a wiggle (WIG) format file for visualization, a sequence file for subsequent motif analysis and an annotation file"
- [readme] Pair-end fragment handling: "Fully supports pair-end data by treating two ends together as ONE fragment"
- [readme] Concrete invocation parameters: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
