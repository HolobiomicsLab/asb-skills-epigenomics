---
name: quorum-sensing-promoter-sequence-analysis
description: Use when a promoter FASTA needs to be scanned for TF binding motifs (e.g.,
  H-NS nucleation sites in luxCDABE) and the predictions must be compared to established
  binding-site coordinates.
when_to_use_negative:
- The question is genome-wide ChIP-seq peak overlap or annotation rather than motif
  scanning of a single promoter — use the repository's Perl scripts (findOverlaps.pl,
  getPeakAnno.pl) on MACS2 peaks instead
- No reference promoter sequence or motif is available and cannot be retrieved from
  the deposited resources
- A quantitative measure of binding or expression (RNA-seq, ChIP-qPCR) is needed;
  motif occurrence alone is only a putative-site prediction
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0797
tools:
- name: FIMO
  role: Scans the luxCDABE promoter FASTA with the MEME-format H-NS motif to enumerate
    putative binding sites with default significance scoring
- name: Perl
  role: Helper scripts in the deposited repository reformat FASTA files and identify
    overlapping features for the comparison against LuxR sites
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:9c25732da1fb594a0e509e9b56883cada589fdbcc264206387f731896501a283
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/Juliacvk/LuxR_H-NS
  repo_url: https://github.com/Juliacvk/LuxR_H-NS
  tools:
  - FIMO
  - Perl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# quorum-sensing-promoter-sequence-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scans a quorum-sensing promoter sequence (e.g., the V. harveyi luxCDABE promoter) for putative transcription-factor binding motifs using FIMO with a curated MEME-format motif, to test whether predicted sites coincide with known regulator binding sites such as LuxR.

## When to use

Use when a promoter FASTA needs to be scanned for TF binding motifs (e.g., H-NS nucleation sites in luxCDABE) and the predictions must be compared to established binding-site coordinates.

## When NOT to use

- The question is genome-wide ChIP-seq peak overlap or annotation rather than motif scanning of a single promoter — use the repository's Perl scripts (findOverlaps.pl, getPeakAnno.pl) on MACS2 peaks instead
- No reference promoter sequence or motif is available and cannot be retrieved from the deposited resources
- A quantitative measure of binding or expression (RNA-seq, ChIP-qPCR) is needed; motif occurrence alone is only a putative-site prediction

## Inputs

- Promoter nucleotide sequence in FASTA format (e.g., luxCDABE promoter from the deposited V. harveyi sequence data)
- MEME-format motif file encoding the H-NS nucleation motif TCGATAAATT
- Reference binding-site coordinates (e.g., the seven established LuxR binding sites in PluxCDABE)

## Outputs

- FIMO tabular report of predicted motif matches with genomic coordinates and significance scores
- Count and list of putative H-NS binding sites (expected: 13 in PluxCDABE)
- Overlap annotation mapping predicted H-NS sites onto the seven established LuxR binding sites

## How to apply

Retrieve the promoter nucleotide sequence from the deposited V. harveyi resources in the github.com/Juliacvk/LuxR_H-NS repository, which also carries the authors' Perl scripts for peak-overlap analysis. Construct a MEME-format motif file encoding the high-affinity E. coli H-NS nucleation motif TCGATAAATT, then run FIMO against the promoter FASTA with default significance scoring to enumerate motif matches and coordinates. Tabulate the predicted site coordinates and compare them against the reference annotation — here, the seven established LuxR binding sites in PluxCDABE. Validation is done by checking that the predicted site count and overlap pattern reproduce the published result of 13 putative H-NS sites overlapping the LuxR sites.

## Related tools

- **FIMO** (Scans the luxCDABE promoter FASTA with the MEME-format H-NS motif to enumerate putative binding sites with default significance scoring)
- **Perl** (Helper scripts in the deposited repository reformat FASTA files and identify overlapping features for the comparison against LuxR sites) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
fimo TCGATAAATT.meme luxCDABE_promoter.fasta
```

## Evaluation signals

- FIMO on the PluxCDABE promoter with the TCGATAAATT motif must return exactly 13 putative H-NS binding sites
- Predicted site coordinates must positionally overlap the seven established LuxR binding sites as annotated in the paper
- The motif file must faithfully encode the 10-bp H-NS nucleation motif TCGATAAATT in MEME format
- FIMO output rows must carry coordinates and significance scores consistent with default scoring parameters
- Sequence retrieval must be traceable to the deposited V. harveyi resources in the github.com/Juliacvk/LuxR_H-NS repository

## Limitations

- FIMO hits are putative motif matches only — they indicate sequence similarity to the H-NS nucleation motif, not verified H-NS occupancy (the paper validates occupancy by ChIP-seq/ChIP-PCR)
- The motif TCGATAAATT is the high-affinity E. coli H-NS nucleation motif, so lower-affinity or V. harveyi-specific H-NS sites may be missed
- Promoter-specific regulation varies: the paper notes 'the transcriptional control of each QS promoter is likely different', so luxCDABE findings may not generalize to other promoters

## Evidence

- [other] FIMO scan identifies 13 putative H-NS sites overlapping LuxR sites: [evidence span withheld — anchor status: not_found]
