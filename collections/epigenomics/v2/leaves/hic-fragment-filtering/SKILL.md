---
name: hic-fragment-filtering
description: Use when preparing Hi-C fragment files (e.g. from extractHAIRS --hic
  1) for haplotype assembly and you need to drop putative trans-homolog linkage errors
  before phasing with SpecHap.
when_to_use_negative:
- Input fragments are from WGS, PacBio SMRT, ONP, or 10X linked-reads rather than
  proximity-ligation (Hi-C) data — the 40 Mbp insertion-size cutoff is specific to
  Hi-C trans-interaction error
- Variants/fragments target a non-diploid organism — SpecHap's graph bisection model
  does not fit increased ploidy, so downstream phasing (and hence this filtering step
  feeding it) is inappropriate
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: HapCUT2
  role: Provides extractHAIRS (--hic 1 flag) used to generate the Hi-C fragment file
    from the aligned BAM and VCF before filtering
  repo: https://github.com/vibansal/HapCUT2
- name: SpecHap
  role: Downstream phasing tool whose Hi-C input format (fragment file sorted on column
    6) and error model motivate the 40 Mbp trans-homolog filter
  repo: https://github.com/deepomicslab/SpecHap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:f77ca38f5e8c46971265d6e993a2ffb7c74948a80c1196a0b88b531ac3a79dce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/deepomicslab/SpecHap
  repo_url: https://github.com/vibansal/HapCUT2
  tools:
  - HapCUT2
  - SpecHap
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hic-fragment-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters Hi-C fragment files prior to haplotype assembly by removing read pairs whose inferred insertion size spans more than 40 Mbp, thereby excluding trans interactions between homologous chromosomes that SpecHap treats as errors rather than modeling explicitly.

## When to use

Use when preparing Hi-C fragment files (e.g. from extractHAIRS --hic 1) for haplotype assembly and you need to drop putative trans-homolog linkage errors before phasing with SpecHap.

## When NOT to use

- Input fragments are from WGS, PacBio SMRT, ONP, or 10X linked-reads rather than proximity-ligation (Hi-C) data — the 40 Mbp insertion-size cutoff is specific to Hi-C trans-interaction error
- Variants/fragments target a non-diploid organism — SpecHap's graph bisection model does not fit increased ploidy, so downstream phasing (and hence this filtering step feeding it) is inappropriate

## Inputs

- Aligned Hi-C BAM file
- VCF file with variant calls for a single diploid individual
- Sorted Hi-C fragment file produced by extractHAIRS (HapCUT2) with --hic 1, sorted numerically on column 6

## Outputs

- hic_fragments_filtered.frag — fragment-format file with no fragment spanning more than 40 Mbp inferred insertion size

## How to apply

First generate the Hi-C fragment file by running extractHAIRS from HapCUT2 on the aligned BAM and VCF with the --hic 1 flag, since SpecHap's pipeline uses a modified ExtractHair from HapCUT2 for fragment processing. Sort the resulting fragment file numerically by column 6 using 'sort -n -k6', the ordering required for Hi-C fragment input. Then parse each fragment record, compute the inferred insertion size between the paired read loci, and discard any fragment whose inferred insertion exceeds 40 Mbp — the threshold chosen to exclude trans-homolog interactions, which carry higher trans interaction error rates and are treated as errors rather than modeled by SpecHap. Write the surviving fragments in the same fragment format to hic_fragments_filtered.frag.

## Related tools

- **HapCUT2** (Provides extractHAIRS (--hic 1 flag) used to generate the Hi-C fragment file from the aligned BAM and VCF before filtering) — https://github.com/vibansal/HapCUT2
- **SpecHap** (Downstream phasing tool whose Hi-C input format (fragment file sorted on column 6) and error model motivate the 40 Mbp trans-homolog filter) — https://github.com/deepomicslab/SpecHap

## Examples

```
extractHAIRS --hic 1 --bam aln.bam --vcf variants.vcf > fragments.frag && sort -n -k6 fragments.frag | awk '{if ($6 - $4 <= 40000000) print}' > hic_fragments_filtered.frag
```

## Evaluation signals

- hic_fragments_filtered.frag exists and retains the sorted (sort -n -k6) fragment format expected for Hi-C input
- No fragment in the output has an inferred insertion size larger than 40 Mbp
- All output records remain in the original fragment format produced by extractHAIRS
- Downstream SpecHap runs accept the filtered file without fragment-format errors
- Filter rate is modest: only fragments spanning inter-locus distances >40 Mbp (putative trans-homolog links) are removed, not within-chromosome pairs

## Limitations

- SpecHap treats trans-homolog interactions as general errors rather than modeling them specifically, so genuinely informative trans contacts are discarded by this filter
- A comprehensive analysis of the influence of technology-specific error (including Hi-C) on phasing quality has not yet been conducted, so the 40 Mbp threshold is heuristic
- SpecHap supports only SNPs, short insertions and deletions; fragments spanning genome rearrangements are not handled by the linkage graph
- No software version is stated for SpecHap, so the filtering behavior may vary across builds

## Evidence
