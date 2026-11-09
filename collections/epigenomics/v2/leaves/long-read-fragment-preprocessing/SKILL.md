---
name: long-read-fragment-preprocessing
description: Use when preparing Hi-C proximity-ligation read data for diploid haplotype
  assembly with SpecHap, and trans-homolog interactions (inferred insertion > 40 Mbp)
  must be removed from fragment input.
when_to_use_negative:
- Input fragments come from PacBio SMRT, ONT, WGS, or 10X linked-reads — the 40 Mbp
  trans-homolog insertion-size cutoff is specific to Hi-C proximity-ligation data
- Fragments have already been filtered for insertion size or were not produced in
  Hi-C mode (--hic 1), so insertion-size inference between paired loci is not applicable
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: Downstream diploid phasing tool that consumes the filtered, sorted Hi-C fragment
    file; the 40 Mbp filter avoids linkages with higher trans interaction error rates
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified ExtractHair utility run with --hic 1 to extract Hi-C
    fragments from the aligned BAM and VCF
  repo: https://github.com/vibansal/HapCUT2
- name: htslib
  role: Required dependency for fragment processing (ExtractHair builds against htslib)
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:039df71d021d687c27d7c1acbceed6bc8e420a8885b3dfc8f7446f3fe7af026b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/deepomicslab/SpecHap
  tools:
  - HapCUT2
  - SpecHap
  - htslib
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# long-read-fragment-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess Hi-C fragment files for SpecHap phasing by generating fragments with extractHAIRS (HapCUT2) in Hi-C mode, numerically sorting them, and filtering out read pairs whose inferred insertion size exceeds 40 Mbp to exclude trans-homolog interactions treated as errors.

## When to use

Use when preparing Hi-C proximity-ligation read data for diploid haplotype assembly with SpecHap, and trans-homolog interactions (inferred insertion > 40 Mbp) must be removed from fragment input.

## When NOT to use

- Input fragments come from PacBio SMRT, ONT, WGS, or 10X linked-reads — the 40 Mbp trans-homolog insertion-size cutoff is specific to Hi-C proximity-ligation data
- Fragments have already been filtered for insertion size or were not produced in Hi-C mode (--hic 1), so insertion-size inference between paired loci is not applicable

## Inputs

- Aligned Hi-C reads in BAM format
- VCF file with variant calls (bgzipped for SpecHap input)
- Hi-C fragment file produced by extractHAIRS with --hic 1

## Outputs

- hic_fragments_filtered.frag — fragment-format file, numerically sorted by column 6, containing no fragment with inferred insertion spanning more than 40 Mbp

## How to apply

Generate the Hi-C fragment file by running extractHAIRS (from HapCUT2) on the aligned BAM and VCF with the --hic 1 flag to enable Hi-C mode. Sort the resulting fragment file numerically by column 6 using 'sort -n -k6', which is the ordering SpecHap requires for Hi-C fragment input. Parse each fragment record, compute the inferred insertion size between the paired read loci, and remove any fragment whose inferred insertion is larger than 40 Mbp — these linkages carry higher trans interaction error rates and are treated as errors rather than modeled specifically. Write surviving fragments in the same fragment format to hic_fragments_filtered.frag, which along with a bgzipped VCF serves as SpecHap input.

## Related tools

- **SpecHap** (Downstream diploid phasing tool that consumes the filtered, sorted Hi-C fragment file; the 40 Mbp filter avoids linkages with higher trans interaction error rates) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified ExtractHair utility run with --hic 1 to extract Hi-C fragments from the aligned BAM and VCF) — https://github.com/vibansal/HapCUT2
- **htslib** (Required dependency for fragment processing (ExtractHair builds against htslib)) — https://github.com/samtools/htslib

## Examples

```
extractHAIRS --hic 1 --vcf variants.vcf.gz --bam aln.bam > fragments.frag && sort -n -k6 fragments.frag > fragments_sorted.frag
```

## Evaluation signals

- hic_fragments_filtered.frag exists and retains the fragment file format
- Output fragment file remains numerically sorted by column 6 (the ordering required for Hi-C fragment input)
- No fragment in the output has an inferred insertion spanning more than 40 Mbp
- Every record in the output traces back to a fragment in the extractHAIRS output (only removals, no format alterations)
- SpecHap accepts the filtered fragment file together with the bgzipped VCF without format errors

## Limitations

- The 40 Mbp cutoff is a heuristic against trans-homolog interaction error; a comprehensive analysis of technology-specific error for Hi-C and 10x linked-reads phasing quality had not yet been conducted
- SpecHap's current graph bisection model supports only SNPs, short insertions and deletions — genome rearrangements are not incorporated in the linkage graph
- SpecHap is diploid only; the bisection model may not fit when ploidy increases

## Evidence

- [methods] SpecHap removes Hi-C read pairs whose inferred insertion size exceeds 40 Mbp, avoiding linkages with higher trans interaction error rates.: [evidence span withheld — anchor status: not_found]
- [supplementary] Sort the fragment file numerically by column 6 using 'sort -n -k6'.: [evidence span withheld — anchor status: not_found]
