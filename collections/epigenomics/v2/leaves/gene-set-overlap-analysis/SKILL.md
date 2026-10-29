---
name: gene-set-overlap-analysis
description: Use when you have two or more differential-expression-derived gene lists
  (e.g., H-NS regulons from GSE136050 RNA-seq and a published 308-gene LuxR regulon)
  and need to compute and validate their overlap count.
when_to_use_negative:
- Input is already a precomputed overlap/Venn result rather than raw gene lists requiring
  identifier harmonization and intersection
- The regulon gene lists were derived with different expression thresholds (not >2-fold,
  P < 0.05), which would change the expected 178-gene overlap count
- You need ChIP-seq peak-level overlap analysis with genomic coordinate logic rather
  than gene-set list intersection (note the README scripts such as findOverlaps.pl
  were built for Macs2 peak overlaps)
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Perl
  role: Used to compute the set intersection of the H-NS and LuxR regulon gene lists
    via the deposited scripts (e.g., findOverlaps.pl, FileUtil.pm)
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:8a308d603fc11df81f50d6170b4203dec3c1a2dd3c3facbe29b3cfdff5096a4a
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gene-set-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the set intersection between two regulon gene lists — here, the RNA-seq-derived H-NS regulons (wild-type and ΔluxR backgrounds) and the previously identified 308-gene LuxR regulon in V. harveyi — to quantify regulatory overlap using the Perl scripts deposited at github.com/Juliacvk/LuxR_H-NS.

## When to use

Use when you have two or more differential-expression-derived gene lists (e.g., H-NS regulons from GSE136050 RNA-seq and a published 308-gene LuxR regulon) and need to compute and validate their overlap count.

## When NOT to use

- Input is already a precomputed overlap/Venn result rather than raw gene lists requiring identifier harmonization and intersection
- The regulon gene lists were derived with different expression thresholds (not >2-fold, P < 0.05), which would change the expected 178-gene overlap count
- You need ChIP-seq peak-level overlap analysis with genomic coordinate logic rather than gene-set list intersection (note the README scripts such as findOverlaps.pl were built for Macs2 peak overlaps)

## Inputs

- H-NS regulon gene lists derived from RNA-seq differential expression (GSE136050; wild-type and ΔluxR backgrounds; >2-fold, P < 0.05)
- Previously identified 308-gene LuxR regulon gene list (reference 7)
- Harmonized gene identifier mapping between the two regulon lists

## Outputs

- hns_luxr_regulon_overlap.csv containing the overlapping genes and the overlap count (expected: 178 genes)

## How to apply

Retrieve the H-NS regulon gene lists from the RNA-seq differential expression comparisons (NCBI GEO accession GSE136050), which used thresholds of >2-fold change and P < 0.05. Load the reference 308-gene LuxR regulon list and harmonize gene identifiers between the two sets before intersecting. Compute the set intersection using the Perl helper scripts deposited at github.com/Juliacvk/LuxR_H-NS (e.g., findOverlaps.pl for overlap/Venn-style set comparisons; FileUtil.pm assists with file loading). Write the overlapping genes and the overlap count to hns_luxr_regulon_overlap.csv. Validate that the computed overlap count equals the reported 178 genes from the 308-gene LuxR regulon.

## Related tools

- **Perl** (Used to compute the set intersection of the H-NS and LuxR regulon gene lists via the deposited scripts (e.g., findOverlaps.pl, FileUtil.pm)) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
python -c "A=set(open('luxr_regulon_308genes.txt')); B=set(open('hns_regulon.txt')); [print(g.strip()) for g in A&B]" > hns_luxr_regulon_overlap.csv
```

## Evaluation signals

- The number of genes in the computed overlap equals the reported 178 genes
- The overlap count does not exceed the size of the smaller gene set (308 genes in the LuxR regulon)
- No unharmonized or unmatched gene identifiers remain in either input list before intersection
- The output file hns_luxr_regulon_overlap.csv contains both the overlapping gene IDs and the overlap count
- Re-running the Perl scripts on the GSE136050-derived inputs reproduces the same intersection count

## Limitations

- The expected 178-gene overlap depends on the fixed differential-expression thresholds (>2-fold, P < 0.05); changing cutoffs changes the result and breaks validation
- Gene identifier mismatches between the H-NS and LuxR regulon lists can inflate or deflate the intersection count
- The README documents the scripts primarily for Macs2 ChIP-seq peak overlaps (findOverlaps.pl, getPeakAnno.pl, getGenePeakAnno2.pl), so adaptation is needed for pure gene-list set operations

## Evidence

- [other] Compute the set intersection of the two regulon gene lists using the Perl scripts deposited at github.com/Juliacvk/LuxR_H-NS.: [evidence span withheld — anchor status: not_found]
- [other] Validation: the number of genes in the computed overlap must equal the reported 178 genes.: [evidence span withheld — anchor status: not_found]
