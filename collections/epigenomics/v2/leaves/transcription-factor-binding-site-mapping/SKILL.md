---
name: transcription-factor-binding-site-mapping
description: Use when you have a promoter nucleotide sequence (FASTA) and a known
  TF consensus/position-specific motif, and need to enumerate motif matches and test
  positional overlap with other factors' binding sites.
when_to_use_negative:
- Input is ChIP-seq peak data rather than a promoter nucleotide sequence — use peak-overlap
  tooling (e.g., the deposited findOverlaps.pl/getPeakAnno.pl scripts) instead
- No credible motif for the factor of interest is available; FIMO scanning of an arbitrary/unsupported
  motif would not reproduce the article's H-NS nucleation-site logic
- The goal is genome-wide binding discovery from ChIP-seq (e.g., the 506 LuxR / 302
  H-NS peaks via MACS2), not targeted motif scanning of a single promoter
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0749
tools:
- name: FIMO
  role: Scans the PluxCDABE promoter FASTA with the MEME-format H-NS nucleation motif
    TCGATAAATT under default significance scoring to enumerate motif matches
- name: Perl
  role: Deposited helper scripts (FileUtil.pm, reformatFasta.pl) clean and reformat
    the multi-FASTA promoter sequence prior to scanning
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:3ded0cebff3d84620b7317d6a1e57221530362ff14bb55a82ba4fc5260b1bc89
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
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# transcription-factor-binding-site-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps transcription factor binding sites on a promoter sequence by scanning a known, high-affinity motif (e.g., the E. coli H-NS nucleation motif TCGATAAATT) with FIMO, then intersecting predicted sites with independently established binding sites (here, the seven established LuxR binding sites in the PluxCDABE promoter).

## When to use

Use when you have a promoter nucleotide sequence (FASTA) and a known TF consensus/position-specific motif, and need to enumerate motif matches and test positional overlap with other factors' binding sites.

## When NOT to use

- Input is ChIP-seq peak data rather than a promoter nucleotide sequence — use peak-overlap tooling (e.g., the deposited findOverlaps.pl/getPeakAnno.pl scripts) instead
- No credible motif for the factor of interest is available; FIMO scanning of an arbitrary/unsupported motif would not reproduce the article's H-NS nucleation-site logic
- The goal is genome-wide binding discovery from ChIP-seq (e.g., the 506 LuxR / 302 H-NS peaks via MACS2), not targeted motif scanning of a single promoter

## Inputs

- luxCDABE promoter nucleotide sequence in FASTA format (from the deposited V. harveyi resources in github.com/Juliacvk/LuxR_H-NS)
- MEME-format motif file encoding the high-affinity E. coli H-NS nucleation motif TCGATAAATT
- Optionally, a coordinate table of established LuxR binding sites (seven sites) in the PluxCDABE promoter

## Outputs

- FIMO match table enumerating 13 putative H-NS binding sites with coordinates and significance scores on the PluxCDABE promoter
- Overlap annotation indicating which predicted H-NS sites coincide with the seven established LuxR binding sites

## How to apply

Retrieve the luxCDABE promoter nucleotide sequence from the deposited V. harveyi resources in the github.com/Juliacvk/LuxR_H-NS repository, cleaning multi-FASTA input with the deposited Perl script reformatFasta.pl if needed. Construct a MEME-format motif file encoding the high-affinity E. coli H-NS nucleation motif TCGATAAATT. Run FIMO against the promoter FASTA with default significance scoring to enumerate motif matches. Tabulate the predicted H-NS binding site coordinates and compare them positionally against the seven established LuxR binding sites in the PluxCDABE promoter to identify coinciding sites. The rationale is that H-NS oligomerizes on DNA to form filaments, so nucleation-motif density overlapping LuxR sites supports the LuxR/H-NS co-occupancy mechanism reported for 28 co-regulated promoters.

## Related tools

- **FIMO** (Scans the PluxCDABE promoter FASTA with the MEME-format H-NS nucleation motif TCGATAAATT under default significance scoring to enumerate motif matches)
- **Perl** (Deposited helper scripts (FileUtil.pm, reformatFasta.pl) clean and reformat the multi-FASTA promoter sequence prior to scanning) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
fimo --thresh 1e-4 hns_TCGATAAATT.meme luxCDABE_promoter.fa > fimo_hns_sites.tsv
```

## Evaluation signals

- The number of FIMO-predicted H-NS sites on the PluxCDABE promoter must equal the reported 13 putative H-NS binding sites
- Positional overlap of predicted sites against the seven established LuxR binding sites must match the paper's annotation (many sites overlap)
- The motif file must parse as valid MEME format and encode the exact nucleation motif TCGATAAATT
- Promoter FASTA must match the sequence deposited in the Juliacvk/LuxR_H-NS repository resources; reformatFasta.pl should run cleanly on the input
- FIMO runs should use default significance scoring; changing p-value thresholds should break agreement with the 13-site count

## Limitations

- The motif TCGATAAATT is the high-affinity E. coli H-NS nucleation motif applied to V. harveyi; cross-species motif transfer may miss sites or report false matches
- Predicted motif matches are sequence-based only and do not by themselves demonstrate H-NS occupancy; ChIP-seq/ChIP-PCR evidence is separate
- Results depend on the promoter sequence deposited with the repository (a JBrowse session hosts the uploaded sequences); sequence/version mismatches will shift coordinates
- The paper notes other regulatory mechanisms repress luxCDABE at LCD even in Δhns, so motif mapping alone does not explain full regulation

## Evidence

- [other] FIMO scan workflow: [evidence span withheld — anchor status: not_found]
