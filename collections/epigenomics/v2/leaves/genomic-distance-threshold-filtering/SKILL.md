---
name: genomic-distance-threshold-filtering
description: Use when preparing Hi-C fragment files for haplotype assembly with SpecHap
  and you need to drop read pairs spanning more than 40 Mbp to avoid linkages with
  elevated trans interaction error rates.
when_to_use_negative:
- Input fragments come from WGS, PacBio SMRT, ONT, or 10X linked-reads rather than
  Hi-C — the 40 Mbp insertion-size cutoff targets Hi-C trans-homolog error, not long-read
  or linked-read data
- Fragments have not yet been sorted by 'sort -n -k6'; SpecHap requires the Hi-C fragment
  ordering before filtering/phasing
- The downstream phasing tool models trans interactions explicitly rather than treating
  them as errors
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: SpecHap
  role: Downstream diploid phasing algorithm; its error model treats trans-homolog
    Hi-C interactions as errors, motivating the 40 Mbp distance filter
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Provides extractHAIRS, used with --hic 1 to extract Hi-C fragment files from
    the aligned BAM and VCF prior to filtering
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:2bfd1454b7fd817af1f0bf078b331de0616ae151943ba4175ccf2828df027b00
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Genomic distance threshold filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters Hi-C fragment records by removing read pairs whose inferred insertion size between the paired read loci exceeds 40 Mbp, excluding trans-homolog interactions that SpecHap treats as generic errors rather than modeling them explicitly.

## When to use

Use when preparing Hi-C fragment files for haplotype assembly with SpecHap and you need to drop read pairs spanning more than 40 Mbp to avoid linkages with elevated trans interaction error rates.

## When NOT to use

- Input fragments come from WGS, PacBio SMRT, ONT, or 10X linked-reads rather than Hi-C — the 40 Mbp insertion-size cutoff targets Hi-C trans-homolog error, not long-read or linked-read data
- Fragments have not yet been sorted by 'sort -n -k6'; SpecHap requires the Hi-C fragment ordering before filtering/phasing
- The downstream phasing tool models trans interactions explicitly rather than treating them as errors

## Inputs

- Aligned Hi-C BAM file
- VCF file with variant calls
- Hi-C fragment file produced by extractHAIRS with --hic 1, sorted numerically by column 6

## Outputs

- hic_fragments_filtered.frag containing fragments with inferred insertion size <= 40 Mbp, retaining the sorted fragment format

## How to apply

First generate the Hi-C fragment file by running extractHAIRS (from HapCUT2) on the aligned BAM and VCF with the --hic 1 flag, then numerically sort the fragment file by column 6 with 'sort -n -k6' as required for Hi-C fragment input. Parse each fragment record and compute the inferred insertion size between the two paired-read loci; remove any fragment whose inferred insertion spans more than 40 Mbp (40,000,000 bases). Write surviving fragments in the same fragment format to the output file. The rationale is that trans interactions between homologous chromosomes are treated as errors in the SpecHap linkage graph, so distal pairs with higher trans error rates are removed before phasing.

## Related tools

- **SpecHap** (Downstream diploid phasing algorithm; its error model treats trans-homolog Hi-C interactions as errors, motivating the 40 Mbp distance filter) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Provides extractHAIRS, used with --hic 1 to extract Hi-C fragment files from the aligned BAM and VCF prior to filtering) — https://github.com/vibansal/HapCUT2

## Examples

```
sort -n -k6 hic_fragments.frag | awk '{if ($6 - $2 <= 40000000) print}' > hic_fragments_filtered.frag
```

## Evaluation signals

- hic_fragments_filtered.frag exists after the filtering step
- The output retains the fragment format and the numeric sort by column 6
- No fragment in the output has an inferred insertion size larger than 40 Mbp (e.g. verify max(column6 span) <= 40,000,000)
- Downstream SpecHap runs accept the filtered fragment file as Hi-C input without format errors

## Limitations

- The 40 Mbp cutoff removes genuine inter-chromosomal/trans-homolog linkages wholesale rather than modeling them, which may discard real signal
- The article notes a comprehensive analysis of technology-specific error (including Hi-C) over phasing quality has not yet been conducted
- SpecHap provides no guarantee of optimal solutions and phasing results depend on the interval size parameter

## Evidence

- [other] trans-homolog filter rationale: [evidence span withheld — anchor status: not_found]
- [other] extractHAIRS Hi-C mode: [evidence span withheld — anchor status: not_found]
- [other] sorting requirement: [evidence span withheld — anchor status: not_found]
