---
name: hns-nucleation-site-prediction
description: Use when a promoter or intergenic FASTA must be scanned for H-NS nucleation
  motifs to test whether H-NS binding sites overlap established LuxR binding sites,
  as for PluxCDABE.
when_to_use_negative:
- Input is ChIP-seq peak data needing overlap or gene annotation — use the repository's
  findOverlaps.pl, getPeakAnno.pl, or getGenePeakAnno2.pl instead of a motif scan.
- The goal is to test whether LuxR evicts H-NS from promoter DNA in vivo or in vitro
  — this requires the ChIP-PCR or in vitro eviction assays, not motif prediction.
- Scanning for LuxR binding sites rather than H-NS nucleation sites — a different
  motif must be used; the TCGATAAATT H-NS motif would give wrong predictions.
edam_topics:
- http://edamontology.org/topic_3511
tools:
- name: FIMO
  role: scans the MEME-format TCGATAAATT H-NS nucleation motif against the PluxCDABE
    promoter FASTA with default significance scoring
- name: Perl
  role: deposited scripts (e.g. reformatFasta.pl) clean and reformat the multi-FASTA
    promoter sequences used as scan input
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:fa8699db0de3deba2d34222950b5f0abc16a8bef63e26f9a0148147b8bdcc20d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# hns-nucleation-site-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In silico prediction of putative H-NS binding sites in a promoter by scanning a MEME-format motif (the high-affinity E. coli H-NS nucleation motif TCGATAAATT) with FIMO, then comparing predicted site coordinates against known regulator binding sites such as the seven established LuxR sites in the luxCDABE promoter.

## When to use

Use when a promoter or intergenic FASTA must be scanned for H-NS nucleation motifs to test whether H-NS binding sites overlap established LuxR binding sites, as for PluxCDABE.

## When NOT to use

- Input is ChIP-seq peak data needing overlap or gene annotation — use the repository's findOverlaps.pl, getPeakAnno.pl, or getGenePeakAnno2.pl instead of a motif scan.
- The goal is to test whether LuxR evicts H-NS from promoter DNA in vivo or in vitro — this requires the ChIP-PCR or in vitro eviction assays, not motif prediction.
- Scanning for LuxR binding sites rather than H-NS nucleation sites — a different motif must be used; the TCGATAAATT H-NS motif would give wrong predictions.

## Inputs

- luxCDABE promoter nucleotide sequence as FASTA (from the V. harveyi resources deposited with github.com/Juliacvk/LuxR_H-NS)
- MEME-format motif file encoding the high-affinity H-NS nucleation motif TCGATAAATT

## Outputs

- FIMO table of predicted H-NS binding site coordinates and significance scores
- Overlap annotation mapping each predicted H-NS site to the seven established LuxR binding sites

## How to apply

Retrieve the luxCDABE promoter nucleotide sequence from the deposited V. harveyi resources accompanying the github.com/Juliacvk/LuxR_H-NS repository, cleaning multi-FASTA inputs with the deposited reformatFasta.pl Perl script if needed. Build a MEME-format motif file encoding the high-affinity E. coli H-NS nucleation motif TCGATAAATT. Run FIMO against the promoter FASTA with default significance scoring to enumerate motif matches with coordinates. Tabulate the predicted H-NS binding site coordinates and annotate which coincide with the seven established LuxR binding sites, since the paper shows LuxR counter-silences by competing with H-NS at co-occupied promoters.

## Related tools

- **FIMO** (scans the MEME-format TCGATAAATT H-NS nucleation motif against the PluxCDABE promoter FASTA with default significance scoring)
- **Perl** (deposited scripts (e.g. reformatFasta.pl) clean and reformat the multi-FASTA promoter sequences used as scan input) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
fimo --verbosity 1 TCGATAAATT_hns.meme luxCDABE_promoter.fasta
```

## Evaluation signals

- Number of predicted H-NS sites must equal the reported 13 putative H-NS binding sites in the luxCDABE promoter.
- Positional overlap of predicted sites against the seven established LuxR binding sites must match the paper's annotation.
- Motif file must encode the exact 10-mer TCGATAAATT in MEME format before running FIMO.
- FIMO must be run with default significance scoring (no custom p-value thresholds introduced).
- Reported coordinates must be traceable back to the promoter FASTA from the deposited repository sequence data.

## Limitations

- The scan uses the single high-affinity E. coli H-NS nucleation motif TCGATAAATT, which may not capture all H-NS oligomerization/bridging binding modes.
- Motif-site overlap with LuxR sites is computational evidence only; co-occupancy claims in the paper rest on ChIP-seq/ChIP-PCR (28 co-occupied promoters), so overlaps should not be interpreted as direct occupancy.
- The paper notes that transcriptional control of each QS promoter is likely different, so luxCDABE-specific site counts may not generalize to other promoters.

## Evidence

- [results] A FIMO scan of the PluxCDABE promoter using the high-affinity E. coli H-NS nucleation motif TCGATAAATT identified 13 putative H-NS binding sites: [evidence span withheld — anchor status: not_found]
