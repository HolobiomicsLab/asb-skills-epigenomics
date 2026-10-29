---
name: hdf-bin-count-generation
description: Use when raw ChIP-Seq short reads (BED/ELAND, single- or pair-end) must
  be converted into per-bin HDF counts to build a genome-wide coverage profile for
  peak calling or count-distribution modeling.
when_to_use_negative:
- When the goal is peak calling directly from already-computed coverage bigWig files
  rather than from raw short reads needing directional HDF extension
- When data are comparative histone modification tracks that ChIPDiff-style methods
  handle, since HPeak targets protein-DNA binding site ChIP-Seq data
- When reads of different orientation within a peak must be distinguished to pinpoint
  summit location — the current HPeak algorithm does not use orientation information
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3169
tools:
- name: HPeak
  role: Generates the HDF coverage profile by directionally extending short reads,
    binning the genome into 25 bp bins, counting HDFs per bin, and feeding the profile
    to its two-state HMM with GP/ZIP emissions
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Implementation language of the HPeak software package used to run the bin-count
    generation workflow
- name: C++
  role: Implementation language of the HPeak software package used to run the bin-count
    generation workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:07c5ae56a09044bd4ce058e7fe89405f928076f77ddc53b84e7bced892e7d3fc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C++
  - HPeak
  - Perl
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# hdf-bin-count-generation

## Summary

Generates hypothetical DNA fragment (HDF) bin counts from ChIP-Seq short reads by directionally extending each read and tallying HDFs per fixed 25 bp genomic bin, producing the coverage profile that HPeak's two-state HMM consumes for enriched-region calling and distribution fitting.

## When to use

Use when raw ChIP-Seq short reads (BED/ELAND, single- or pair-end) must be converted into per-bin HDF counts to build a genome-wide coverage profile for peak calling or count-distribution modeling.

## When NOT to use

- When the goal is peak calling directly from already-computed coverage bigWig files rather than from raw short reads needing directional HDF extension
- When data are comparative histone modification tracks that ChIPDiff-style methods handle, since HPeak targets protein-DNA binding site ChIP-Seq data
- When reads of different orientation within a peak must be distinguished to pinpoint summit location — the current HPeak algorithm does not use orientation information

## Inputs

- ChIP-Seq treatment reads in BED (or ELAND) format, single-end or pair-end
- Control ChIP-Seq reads in BED format
- Read length parameter (e.g. -r 36) and fragment-length bounds (-fmin 100, -fmax 300) or pair-end insert size (-isize 200)
- Species/genome specification (HUMAN or MOUSE)

## Outputs

- Per-bin HDF count vector over 25 bp genome-wide bins (HDF coverage profile)
- WIG format coverage file for visualization (-wig)
- FASTA sequence file of enriched regions for downstream motif analysis (-seq)
- Peak annotation file with gene symbol, refseq ID, summit distance, and intermediate files (-interfiles)

## How to apply

Extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF), following the Robertson et al. approach, using the read length (-r, e.g. 36) and fragment-length bounds (-fmin 100, -fmax 300 for single-end) or the insert size (-isize 200 for pair-end, where the two ends are treated together as ONE fragment). Partition the entire genome into fixed-length bins of 25 bp and count the numbers of HDFs falling in each bin. Down-weight the HDF coverage contribution in bins covered by the end portion of each HDF (unbalanced weighting scheme) to account for uncertainty in actual fragment lengths. Run the count generation via HPeak3 (Perl/C++) with -sp HUMAN/MOUSE, -format BED, treatment and control inputs, and emit intermediate files with -interfiles plus WIG (-wig) and FASTA sequence (-seq) outputs. The resulting HDF coverage profile is the input to HPeak's two-state HMM with GP and ZIP emissions, which classifies bins as ChIP-enriched or background; the same counts can subsequently be fit with GP, Poisson, NB, or ZIP distributions by maximum likelihood.

## Related tools

- **HPeak** (Generates the HDF coverage profile by directionally extending short reads, binning the genome into 25 bp bins, counting HDFs per bin, and feeding the profile to its two-state HMM with GP/ZIP emissions) — github.com/cauyrd/HPeak
- **Perl** (Implementation language of the HPeak software package used to run the bin-count generation workflow)
- **C++** (Implementation language of the HPeak software package used to run the bin-count generation workflow)

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles
```

## Evaluation signals

- Intermediate files from -interfiles contain per-bin HDF counts binned at the expected 25 bp fixed length
- Resulting HMM classification separates blocks of consecutive ChIP-enriched bins from background using GP/ZIP emissions as expected
- Coverage contribution is visibly down-weighted in bins covered only by HDF end portions, consistent with the unbalanced weighting scheme
- Pair-end inputs are counted as one fragment per pair rather than two independent reads, matching the -isize handling
- Downstream fit of GP (enriched bins) or ZIP (background bins) to the produced counts converges and outperforms plain Poisson, matching reported behavior

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, though such information helps pinpoint the summit; planned for future release
- Little information on true positive and true negative loci makes performance criteria on experimental ChIP-Seq data hard to determine; results should be interpreted with caution
- Fragment-length uncertainty requires the unbalanced end-weighting scheme, so misspecified -fmin/-fmax or read length (-r) will distort the HDF coverage profile

## Evidence

- [discussion] HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF): "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [readme] Fully supports pair-end data by treating two ends together as ONE fragment: "Fully supports pair-end data by treating two ends together as ONE fragment"
