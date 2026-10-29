---
name: promoter-motif-scanning
description: Use when a promoter FASTA must be scanned for putative binding sites
  of a known motif (e.g., H-NS TCGATAAATT in PluxCDABE) and site coordinates must
  be compared against established binding-site annotations for overlap.
when_to_use_negative:
- The goal is genome-wide ChIP-seq peak calling or peak overlap analysis (506 LuxR
  / 302 H-NS peaks) rather than motif scanning of a single promoter sequence
- No curated or experimentally derived motif is available; de novo motif discovery
  (e.g., MEME) would be required instead of FIMO scanning
- The promoter sequence cannot be retrieved or verified from the deposited resources,
  so site counts cannot be validated against the reported 13 sites
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0749
tools:
- name: FIMO
  role: Scans the luxCDABE promoter FASTA with the MEME-format H-NS nucleation motif
    (TCGATAAATT) to enumerate putative binding sites with default significance scoring
- name: Perl
  role: Repository-deposited Perl scripts support file reformatting (reformatFasta.pl)
    and overlap identification between predicted sites and established binding sites
    (findOverlaps.pl)
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:53a4d2eae610df55e5e34325bed176fe77b21c2292736a581ee2f13a7b03b836
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

# promoter-motif-scanning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scans a promoter nucleotide sequence (e.g., the V. harveyi luxCDABE promoter) for occurrences of a known transcription factor binding motif, such as the high-affinity E. coli H-NS nucleation motif TCGATAAATT, using FIMO with a MEME-format motif file, and compares predicted sites to established binding sites (e.g., the seven LuxR binding sites) for overlap.

## When to use

Use when a promoter FASTA must be scanned for putative binding sites of a known motif (e.g., H-NS TCGATAAATT in PluxCDABE) and site coordinates must be compared against established binding-site annotations for overlap.

## When NOT to use

- The goal is genome-wide ChIP-seq peak calling or peak overlap analysis (506 LuxR / 302 H-NS peaks) rather than motif scanning of a single promoter sequence
- No curated or experimentally derived motif is available; de novo motif discovery (e.g., MEME) would be required instead of FIMO scanning
- The promoter sequence cannot be retrieved or verified from the deposited resources, so site counts cannot be validated against the reported 13 sites

## Inputs

- luxCDABE promoter nucleotide sequence (FASTA) from the deposited V. harveyi resources in github.com/Juliacvk/LuxR_H-NS
- MEME-format motif file encoding the high-affinity E. coli H-NS nucleation motif TCGATAAATT
- Reference annotation of established LuxR binding sites (seven sites) for overlap comparison

## Outputs

- FIMO hit table of predicted H-NS binding sites with genomic coordinates and significance scores (13 putative sites in PluxCDABE)
- Overlap annotation mapping predicted H-NS sites onto the seven established LuxR binding sites

## How to apply

Retrieve the promoter nucleotide sequence from the deposited resources in the github.com/Juliacvk/LuxR_H-NS repository (Perl scripts plus V. harveyi sequence data). Construct a MEME-format motif file encoding the high-affinity H-NS nucleation motif TCGATAAATT. Run FIMO against the promoter FASTA with default significance scoring to enumerate motif matches and their coordinates. Tabulate predicted sites and compare them against known binding-site annotations (here, the seven established LuxR binding sites) to identify positional overlap; the Perl helper scripts in the repository (e.g., findOverlaps.pl) illustrate the overlap-identification pattern. Validate by checking that the predicted site count matches the reported count (13 putative H-NS sites in PluxCDABE) and that overlaps with LuxR sites reproduce the paper's annotation.

## Related tools

- **FIMO** (Scans the luxCDABE promoter FASTA with the MEME-format H-NS nucleation motif (TCGATAAATT) to enumerate putative binding sites with default significance scoring)
- **Perl** (Repository-deposited Perl scripts support file reformatting (reformatFasta.pl) and overlap identification between predicted sites and established binding sites (findOverlaps.pl)) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
fimo --motif hns_nucleation_TCGATAAATT.meme luxCDABE_promoter.fasta
```

## Evaluation signals

- The FIMO scan of PluxCDABE with the TCGATAAATT motif must return exactly 13 putative H-NS binding sites
- Positional overlap between predicted H-NS sites and the seven established LuxR binding sites must match the paper's annotation
- The motif file must parse as valid MEME format and encode the 10-bp H-NS nucleation motif TCGATAAATT
- Promoter FASTA must match the luxCDABE promoter sequence deposited with the github.com/Juliacvk/LuxR_H-NS resources (reformatFasta.pl can be used to clean/normalize the multi-FASTA)
- Each reported hit must carry FIMO default significance scoring output (p-value/coordinates) suitable for coordinate tabulation

## Limitations

- Motif-based prediction is putative: TCGATAAATT matches do not guarantee H-NS occupancy in vivo; ChIP-seq/ChIP-PCR evidence (28 co-occupied promoters; GSE136050, GSE136191) is the occupancy ground truth
- The article notes transcriptional control of each QS promoter is likely different, so site counts and overlap patterns from luxCDABE may not generalize to other promoters
- The scan relies on the high-affinity E. coli H-NS nucleation motif as a proxy; V. harveyi H-NS specificity may differ

## Evidence
