---
name: chip-seq-genome-binning-and-hdf-counting
description: Use when raw aligned ChIP-Seq reads (BED or ELAND format, single- or
  pair-end) must be converted into a fixed-bin coverage profile prior to peak calling
  or HMM classification.
when_to_use_negative:
- Input is not aligned short-read data (e.g. already a peak list or bin count table)
  — binning and HDF extension would be redundant
- Comparative histone modification data better suited to ChIPDiff, which HPeak's authors
  note is the restricted comparative-chromatin method
- Analysis requires distinguishing reads of different orientation within a peak, which
  HPeak's current algorithm does not support
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Performs directional read extension into HDFs, genome partitioning into 25
    bp bins, and unbalanced-weighting HDF counting as preprocessing to its two-state
    HMM
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Wrapper/scripting layer of the HPeak package that drives the binning pipeline
    (HPeak.pl entry point)
- name: C++
  role: Performance-critical implementation language of the HPeak software package
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:a76958902667cf50bd9388052b60396bd5e272214f0f46fdc340d19f35ca986f
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# chip-seq-genome-binning-and-hdf-counting

## Summary

Constructs a per-bin ChIP-Seq coverage signal by directionally extending short reads into hypothetical DNA fragments (HDFs) following Robertson et al., partitioning the genome into fixed-length 25 bp bins, and counting HDF coverage per bin with HPeak's unbalanced weighting scheme that down-weights fragment-end coverage. This signal is the input to downstream HMM-based classification of ChIP-enriched regions.

## When to use

Use when raw aligned ChIP-Seq reads (BED or ELAND format, single- or pair-end) must be converted into a fixed-bin coverage profile prior to peak calling or HMM classification.

## When NOT to use

- Input is not aligned short-read data (e.g. already a peak list or bin count table) — binning and HDF extension would be redundant
- Comparative histone modification data better suited to ChIPDiff, which HPeak's authors note is the restricted comparative-chromatin method
- Analysis requires distinguishing reads of different orientation within a peak, which HPeak's current algorithm does not support

## Inputs

- Aligned ChIP-Seq treatment reads (BED format, or ELAND; SAM/BAM planned) — single-end via -t or pair-end via -pe
- Optional aligned control sample reads passed via -c
- Read length (-r) and, for pair-end data, insert size (-isize)
- Species/genome specification (-sp HUMAN/MOUSE)

## Outputs

- HDF (hypothetical DNA fragment) extended-read coverage profile
- Per-bin HDF counts over fixed-length 25 bp genome bins
- HDF coverage profile feeding the two-state HMM bin classification
- Downstream WIG file, FASTA sequence file, and annotation file outputs (with -ann -wig -seq -interfiles)

## How to apply

Extend each aligned short read directionally from its start position to form a hypothetical DNA fragment (HDF), following Robertson et al.; for pair-end data treat the two ends together as ONE fragment using the -isize insert size (e.g. 200 bp). Partition the entire genome into fixed-length 25 bp bins and count the number of HDFs falling in each bin. Apply HPeak's novel unbalanced weighting scheme to gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF, which accounts for uncertainty in the actual lengths of DNA fragments and improves boundary definition. Specify read length via -r (e.g. -r 36 for single-end, -r 100 for pair-end) and restrict fragments with -fmin/-fmax (e.g. 100–300 bp). The resulting HDF coverage profile over 25 bp bins feeds HPeak's two-state HMM (GP emission for enriched, ZIP for background states).

## Related tools

- **HPeak** (Performs directional read extension into HDFs, genome partitioning into 25 bp bins, and unbalanced-weighting HDF counting as preprocessing to its two-state HMM) — github.com/cauyrd/HPeak
- **Perl** (Wrapper/scripting layer of the HPeak package that drives the binning pipeline (HPeak.pl entry point))
- **C++** (Performance-critical implementation language of the HPeak software package)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Every 25 bp bin of the input genome partition receives an HDF count / state label with no gaps
- Directionally extended HDFs are bounded by the -fmin/-fmax fragment length window (e.g. 100–300 bp) consistent with the -r read length
- Bins covered only by the end portion of an HDF carry lower weight than centrally covered bins, consistent with the unbalanced weighting scheme
- For pair-end input, each read pair contributes ONE fragment using the -isize value, not two independent HDFs
- Resulting coverage profile, when run through the two-state HMM, yields enriched blocks whose boundaries are sharper than unweighted counting would produce (the stated purpose of the weighting scheme)

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, although such information is informative for pinpointing the peak summit
- Uncertainty in the actual lengths of DNA fragments requires the unbalanced weighting approximation when only single-end read data are available
- SAM and BAM input formats are not yet supported (only BED and ELAND), per the README

## Evidence

- [discussion] Extend each short read directionally to form a hypothetical DNA fragment (HDF): "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] Partition genome into 25 bp bins and count HDFs: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] Unbalanced weighting scheme rationale: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [readme] Pair-end handling as one fragment: "Fully supports pair-end data by treating two ends together as ONE fragment"
- [intro] Purpose of weighting scheme: "By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments."
