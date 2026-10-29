---
name: rna-seq-regulon-gene-list-processing
description: Use when two or more differentially expressed gene sets (e.g. H-NS and
  LuxR regulons from RNA-seq comparisons) must be intersected to count shared regulators/genes,
  with identifiers harmonized first.
when_to_use_negative:
- Input is raw RNA-seq reads or count matrices — the skill operates on already-called
  regulon/differential-expression gene lists, not on read processing or DE calling
  itself.
- The task is ChIP-seq peak overlap (e.g. the 88 overlapping LuxR/H-NS peaks proximal
  to an ORF ≤500 bp) rather than gene-list regulon overlap; that requires a separate
  peak-based workflow.
- The gene sets are not from V. harveyi or identifiers cannot be harmonized to a common
  namespace — the reported 178-gene validation target would not apply.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: Perl
  role: Scripting language for the set-intersection/overlap scripts (findOverlaps.pl,
    FileUtil.pm) used to intersect the H-NS and LuxR regulon gene lists
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:d86efda669cc22d569569c124f9f747f90aabfdc94db7c9102dd7bfb18d3d25c
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
  - Perl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# rna-seq-regulon-gene-list-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Harmonize and intersect RNA-seq-derived regulon gene lists — here, the H-NS regulons from wild-type and ΔluxR V. harveyi RNA-seq against the previously identified 308-gene LuxR regulon — using Perl set-operation scripts to quantify regulatory overlap (178 shared genes).

## When to use

Use when two or more differentially expressed gene sets (e.g. H-NS and LuxR regulons from RNA-seq comparisons) must be intersected to count shared regulators/genes, with identifiers harmonized first.

## When NOT to use

- Input is raw RNA-seq reads or count matrices — the skill operates on already-called regulon/differential-expression gene lists, not on read processing or DE calling itself.
- The task is ChIP-seq peak overlap (e.g. the 88 overlapping LuxR/H-NS peaks proximal to an ORF ≤500 bp) rather than gene-list regulon overlap; that requires a separate peak-based workflow.
- The gene sets are not from V. harveyi or identifiers cannot be harmonized to a common namespace — the reported 178-gene validation target would not apply.

## Inputs

- H-NS regulon gene lists from RNA-seq differential expression (wild-type and ΔluxR backgrounds), GEO accession GSE136050
- Previously identified 308-gene LuxR regulon gene list (reference 7)
- Perl scripts from github.com/Juliacvk/LuxR_H-NS (findOverlaps.pl, FileUtil.pm)

## Outputs

- hns_luxr_regulon_overlap.csv containing overlapping genes and the overlap count (expected: 178 genes)
- Venn-diagram-ready overlap sets as produced by findOverlaps.pl

## How to apply

Retrieve the differential-expression gene lists underlying the RNA-seq comparisons (NCBI GEO GSE136050 for the H-NS regulons in wild-type and ΔluxR backgrounds) and load the reference 308-gene LuxR regulon list. Harmonize gene identifiers/locus tags between the lists so the set intersection is meaningful. Run the Perl scripts deposited at github.com/Juliacvk/LuxR_H-NS (e.g. findOverlaps.pl for overlap-set identification, with FileUtil.pm as the file-loading helper) to compute the set intersection of the regulon lists. The published regulon calls use thresholds of >2-fold change and P < 0.05, so input lists should be filtered to the same cutoffs before intersecting. Write the overlapping genes and their count to hns_luxr_regulon_overlap.csv. Validate by checking that the computed overlap count equals the reported 178 genes for the combined H-NS regulons versus the 308-gene LuxR regulon.

## Related tools

- **Perl** (Scripting language for the set-intersection/overlap scripts (findOverlaps.pl, FileUtil.pm) used to intersect the H-NS and LuxR regulon gene lists) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
git clone https://github.com/Juliacvk/LuxR_H-NS && perl findOverlaps.pl  # intersect H-NS and LuxR regulon lists; write overlap to hns_luxr_regulon_overlap.csv
```

## Evaluation signals

- The computed intersection of the combined H-NS regulons with the 308-gene LuxR regulon must contain exactly 178 genes.
- Output file hns_luxr_regulon_overlap.csv exists, contains the overlapping gene list, and its row count matches the reported overlap count.
- Input gene lists were filtered with the published thresholds (>2-fold, P < 0.05) before intersection.
- All overlapping genes appear in both input lists under a single harmonized identifier scheme (no duplicated or unmatched IDs inflating the count).

## Limitations

- The GitHub scripts were written for this study's file layouts (V. harveyi Macs2 peaks and gene lists via FileUtil.pm); adapting them to other organisms/formats requires reformatting.
- Overlap counts are sensitive to the DE thresholds used; the 178-gene figure holds only for >2-fold, P < 0.05 calls and for the combined wild-type and ΔluxR H-NS regulons.
- Gene-list intersection captures transcriptional co-regulation but does not by itself establish direct co-occupancy, which the paper addresses separately via ChIP-seq (88 overlapping peaks proximal to an ORF ≤500 bp).

## Evidence

- [results] Among the combined genes in the H-NS regulons (wild-type and ΔluxR backgrounds combined), 178 genes overlapped with the previously identified 308-gene LuxR regulon (thresholds: >2-fold, P < 0.05; Figure 2A).: [evidence span withheld — anchor status: not_found]
