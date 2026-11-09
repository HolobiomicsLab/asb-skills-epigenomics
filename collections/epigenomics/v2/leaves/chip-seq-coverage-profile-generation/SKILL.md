---
name: chip-seq-coverage-profile-generation
description: Use when mapped ChIP-Seq short reads (BED or ELAND format, or paired-end
  with known insert size) must be converted into a binned, end-weighted WIG-format
  coverage profile as input to HMM-based peak calling.
when_to_use_negative:
- Input is already a genome-scale coverage profile or feature count table rather than
  mapped short reads
- The goal is comparative histone-modification analysis, for which HPeak notes ChIPDiff
  is the dedicated method
- Per-basepair strand-specific coverage (e.g., for peak-summit refinement) is required,
  since HPeak does not distinguish reads of different orientation within a peak
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: HPeak
  role: Implements directional HDF extension, 25 bp binning, and the unbalanced end-weighting
    scheme; emits the WIG coverage file
  repo: github.com/cauyrd/HPeak
- name: Perl
  role: Driver language for the HPeak package, parsing BED/ELAND alignments and orchestrating
    profile generation
- name: C++
  role: Implementation language used for the computationally intensive parts of the
    HPeak package
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:607659f4d09fe4d5384d6a672f581fc621d608ea08dfc7b754633eab7ad85b70
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# chip-seq-coverage-profile-generation

## Summary

Reconstructs a genome-wide ChIP-Seq coverage profile by directionally extending each short read into a hypothetical DNA fragment (HDF), counting HDFs in 25 bp bins, and applying HPeak's unbalanced weighting scheme that down-weights coverage in the uncertain end region of each fragment.

## When to use

Use when mapped ChIP-Seq short reads (BED or ELAND format, or paired-end with known insert size) must be converted into a binned, end-weighted WIG-format coverage profile as input to HMM-based peak calling.

## When NOT to use

- Input is already a genome-scale coverage profile or feature count table rather than mapped short reads
- The goal is comparative histone-modification analysis, for which HPeak notes ChIPDiff is the dedicated method
- Per-basepair strand-specific coverage (e.g., for peak-summit refinement) is required, since HPeak does not distinguish reads of different orientation within a peak

## Inputs

- Mapped short-read alignment file in BED or ELAND format (treatment, and optionally control) via -t/-pe and -c flags
- Read length (-r, e.g. 36) and fragment size range (-fmin/-fmax, e.g. 100-300) for single-end data
- Insert size (-isize, e.g. 200) for paired-end data
- Genome selection (-sp HUMAN or MOUSE)

## Outputs

- WIG-format weighted HDF coverage profile file (-wig flag)
- Per-bin (25 bp) HDF coverage counts feeding the two-state HMM
- Optional FASTA sequence file and annotation file for downstream motif analysis (-seq, -ann)

## How to apply

Parse the alignment file (BED/ELAND, via Perl) to obtain each read's chromosome, start position, and strand, then directionally extend each read from its start position to a hypothetical DNA fragment (HDF), following the Robertson et al. extension scheme used by HPeak. Partition the genome into fixed-length 25 bp bins and count the HDFs falling in each bin. Apply the unbalanced weighting scheme: bins covered by the certain portion of each HDF get full weight, while coverage contribution is gradually down-weighted across the uncertain end region (e.g., full coverage over 175 bp plus a decaying ramp over the remaining region for size-selected fragments between -fmin 100 and -fmax 300; for paired-end data the two ends are treated together as ONE fragment with -isize). Specify the read length explicitly with -r and the species with -sp HUMAN/MOUSE. Write the weighted per-bin profile as a WIG-format file for visualization and downstream HMM analysis.

## Related tools

- **HPeak** (Implements directional HDF extension, 25 bp binning, and the unbalanced end-weighting scheme; emits the WIG coverage file) — github.com/cauyrd/HPeak
- **Perl** (Driver language for the HPeak package, parsing BED/ELAND alignments and orchestrating profile generation)
- **C++** (Implementation language used for the computationally intensive parts of the HPeak package)

## Examples

```
perl HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPREFIX -fmin 100 -fmax 300 -r 36 -wig
```

## Evaluation signals

- Output WIG file exists and parses with fixed 25 bp bin steps matching the HPeak binning scheme
- All weights in the uncertain-end ramp region lie in [0, 1], with full weight (1) in the certain portion of each HDF
- Extended HDF lengths are consistent with the specified -fmin/-fmax range (single-end) or -isize (paired-end)
- WIG profile loads and visualizes correctly in the UCSC genome browser
- Profile coverage is directional (fragments extend from each read's start according to strand), not symmetric around the read

## Limitations

- The current HPeak algorithm does not distinguish reads of different orientation within a peak, which limits summit pinpointing
- For single-end data, fragment length is uncertain and must be approximated via -fmin/-fmax; only the unbalanced weighting scheme compensates for end uncertainty
- Read length must be supplied explicitly (-r); an incorrect value distorts the HDF extension
- Peak-calling performance on experimental data is hard to evaluate because little information on true positive and true negative loci is available

## Evidence

- [intro] Unbalanced end-weighting rationale: "By using a novel unbalanced weighting scheme, HPeak will account for the uncertainties in the actual lengths of DNA fragments."
- [discussion] Directional HDF extension scheme: "we followed the method used by Robertson et al. [11] to extend each short read directionally from its start position to form a hypothetical DNA fragment (HDF)"
- [discussion] Fixed-length binning of HDFs: "HPeak partitions the entire genome into small bins of fixed length (25 bp in this study) and subsequently counts the numbers of HDFs that fall in each bin"
- [discussion] End down-weighting of coverage contribution: "we gradually down-weight the HDF coverage contribution in bins covered by the end portion of each HDF"
- [readme] Concrete invocation with fragment size and read length parameters: "perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN/MOUSE -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPUTPREFIX -fmin 100 -fmax 300 -r 36 -ann -wig -seq -interfiles"
- [readme] Paired-end fragments treated as one unit: "Fully supports pair-end data by treating two ends together as ONE fragment"
