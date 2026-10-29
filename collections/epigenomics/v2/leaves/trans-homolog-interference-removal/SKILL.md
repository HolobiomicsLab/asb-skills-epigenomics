---
name: trans-homolog-interference-removal
description: 'Use when preparing Hi-C fragment files for SpecHap phasing: read pairs
  with inferred insertion spanning >40 Mbp signal trans-homolog interactions and higher
  trans interaction error rates that should be dropped before assembly.'
when_to_use_negative:
- Input fragments come from WGS, PacBio SMRT or Oxford Nanopore data where long spans
  reflect true read length rather than trans-homolog ligation artifacts
- The analysis goal is to model or study trans-homolog interactions rather than exclude
  them from diploid phasing
- The fragment file is not Hi-C mode output (not generated with extractHAIRS --hic
  1), since the insertion-size criterion is Hi-C specific
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: SpecHap
  role: Downstream diploid phasing algorithm whose spectral-graph assembly consumes
    the filtered fragment file; the 40 Mbp cutoff exists because SpecHap treats trans
    interactions as errors rather than modeling them
  repo: https://github.com/deepomicslab/SpecHap
- name: HapCUT2
  role: Source of the modified extractHAIRS utility used to generate the Hi-C fragment
    file with the --hic 1 flag before filtering
  repo: https://github.com/vibansal/HapCUT2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab709
    title: 'SpecHap: a diploid phasing algorithm based on spectral graph theory'
  source_content_hash: sha256:fa3242facf6276d6470bdfaa6207aa4ef037372a34f072b4291e735b262a54b3
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# trans-homolog-interference-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Hi-C fragment filtering step from the SpecHap pipeline that removes read pairs whose inferred insertion size exceeds 40 Mbp, so that trans-homolog interactions—which SpecHap treats as general errors rather than modeling them—are excluded from haplotype assembly.

## When to use

Use when preparing Hi-C fragment files for SpecHap phasing: read pairs with inferred insertion spanning >40 Mbp signal trans-homolog interactions and higher trans interaction error rates that should be dropped before assembly.

## When NOT to use

- Input fragments come from WGS, PacBio SMRT or Oxford Nanopore data where long spans reflect true read length rather than trans-homolog ligation artifacts
- The analysis goal is to model or study trans-homolog interactions rather than exclude them from diploid phasing
- The fragment file is not Hi-C mode output (not generated with extractHAIRS --hic 1), since the insertion-size criterion is Hi-C specific

## Inputs

- Aligned Hi-C BAM file
- VCF file with variant calls
- Unsorted Hi-C fragment file produced by extractHAIRS with --hic 1

## Outputs

- hic_fragments_filtered.frag — sorted (by column 6) Hi-C fragment file containing no fragment with inferred insertion size >40 Mbp

## How to apply

First generate the Hi-C fragment file by running extractHAIRS (a modified utility from HapCUT2 shipped as a SpecHap submodule) on the aligned BAM and VCF with the --hic 1 flag to enable Hi-C mode. Sort the resulting fragment file numerically by column 6 ('sort -n -k6'), the ordering required for Hi-C fragment input. Then parse each fragment record, compute the inferred insertion size between the paired read loci, and remove any fragment whose inferred insertion is larger than 40 Mbp; this avoids linkages with higher trans interaction error rates, since SpecHap does not model trans-homolog interactions explicitly. Write the surviving fragments in the same fragment format to hic_fragments_filtered.frag and feed this file to SpecHap.

## Related tools

- **SpecHap** (Downstream diploid phasing algorithm whose spectral-graph assembly consumes the filtered fragment file; the 40 Mbp cutoff exists because SpecHap treats trans interactions as errors rather than modeling them) — https://github.com/deepomicslab/SpecHap
- **HapCUT2** (Source of the modified extractHAIRS utility used to generate the Hi-C fragment file with the --hic 1 flag before filtering) — https://github.com/vibansal/HapCUT2

## Examples

```
extractHAIRS --hic 1 --vcf variants.vcf --bam aln.bam > frags.frag; sort -n -k6 frags.frag | python drop_gt40mbp.py > hic_fragments_filtered.frag
```

## Evaluation signals

- hic_fragments_filtered.frag exists after the step
- Output retains the fragment file format sorted numerically by column 6
- No fragment in the output has an inferred insertion spanning more than 40 Mbp
- Output fragment count is less than or equal to the input fragment count, and surviving records are otherwise byte-identical to input records

## Limitations

- Trans-homolog interactions are discarded as general errors rather than modeled specifically, so genuine trans-homolog signal is lost
- A comprehensive analysis of the influence of technology-specific error (including Hi-C) over phasing quality has not yet been conducted
- The 40 Mbp insertion-size cutoff is a proxy for trans-homolog events and is applied only to Hi-C fragments in the SpecHap preprocessing workflow

## Evidence

- [other] To exclude trans-homolog interactions treated as general errors, SpecHap removes Hi-C read pairs whose inferred insertion size exceeds 40 Mbp: [evidence span withheld — anchor status: not_found]
- [other] Sort requirement for Hi-C fragment input: [evidence span withheld — anchor status: not_found]
