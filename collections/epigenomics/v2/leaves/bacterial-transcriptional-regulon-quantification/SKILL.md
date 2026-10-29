---
name: bacterial-transcriptional-regulon-quantification
description: Use when RNA-seq count matrices from a regulator-deletion versus control
  strain are available and the goal is to enumerate the regulon (repressed/derepressed
  gene counts) and confirm specific loci such as luxCDABE are recovered.
when_to_use_negative:
- Input is ChIP-seq peak data only (e.g., LuxR/H-NS Macs2 peaks) with no RNA-seq count
  matrix — peak overlap/annotation scripts alone cannot call a regulon.
- The regulon of interest is post-transcriptionally controlled or the signal is below
  the >2-fold, P < 0.05 cutoff — the paper notes luxCDABE has additional repression
  layers at LCD beyond H-NS, so counts alone may miss such genes.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3308
tools:
- name: Perl
  role: Hosts the deposited read-processing, counting, and differential expression
    scripts used to build the count matrix and call H-NS regulon genes at the >2-fold,
    P < 0.05 threshold
  repo: https://github.com/Juliacvk/LuxR_H-NS
- name: getGenePeakAnno2.pl
  role: Perl helper from the same repository that links regulatory peaks to gene information,
    useful for annotating regulon genes
  repo: https://github.com/Juliacvk/LuxR_H-NS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz1089
    title: Global H-NS counter-silencing by LuxR activates quorum sensing gene expression
  source_content_hash: sha256:c76aac0e154fe17fefbc7be0ecb152fc8c681f52068cedeec1ef93f360582b23
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
  - getGenePeakAnno2.pl
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# bacterial-transcriptional-regulon-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the size and composition of a bacterial transcriptional regulon from RNA-seq gene-level count matrices by calling differentially expressed genes between wild-type and regulator-deletion strains (>2-fold change, P < 0.05), retaining direction of change (repressed vs derepressed). Applied here to define the H-NS regulon of V. harveyi at OD600 = 1.0 in wild-type versus ΔluxR backgrounds using the authors' deposited Perl scripts.

## When to use

Use when RNA-seq count matrices from a regulator-deletion versus control strain are available and the goal is to enumerate the regulon (repressed/derepressed gene counts) and confirm specific loci such as luxCDABE are recovered.

## When NOT to use

- Input is ChIP-seq peak data only (e.g., LuxR/H-NS Macs2 peaks) with no RNA-seq count matrix — peak overlap/annotation scripts alone cannot call a regulon.
- The regulon of interest is post-transcriptionally controlled or the signal is below the >2-fold, P < 0.05 cutoff — the paper notes luxCDABE has additional repression layers at LCD beyond H-NS, so counts alone may miss such genes.

## Inputs

- RNA-seq raw reads (FASTQ) for wild-type, Δhns, ΔluxR, and ΔluxR Δhns V. harveyi samples at OD600 = 1.0 (GEO GSE136191)
- V. harveyi reference genome for read mapping
- Authors' Perl analysis scripts (github.com/Juliacvk/LuxR_H-NS) for read processing, counting, and differential expression

## Outputs

- Gene-level count matrix for all four strains at OD600 = 1.0
- H-NS regulon gene list for wild-type versus Δhns with direction of change (repressed vs derepressed)
- H-NS regulon gene list for ΔluxR versus ΔluxR Δhns with direction of change
- luxC fold-change estimate from the wild-type versus Δhns comparison
- Summary of regulon sizes (total and repressed gene counts) for both comparisons

## How to apply

Download RNA-seq raw reads for all four strains (wild-type, Δhns, ΔluxR, ΔluxR Δhns) grown to OD600 = 1.0 from NCBI GEO accession GSE136191, then set up the read-processing and counting pipeline from the Perl scripts deposited at github.com/Juliacvk/LuxR_H-NS. Map reads to the V. harveyi reference genome and generate a gene-level count matrix covering all four strains at OD600 = 1.0. Run the differential expression comparison of wild-type versus Δhns with the deposited Perl scripts to call the H-NS regulon, retaining genes with direction of change (repressed vs derepressed by H-NS), using the >2-fold, P < 0.05 cutoff. Repeat for the ΔluxR versus ΔluxR Δhns comparison to quantify the regulon in the ΔluxR background. Extract the luxC fold change from the wild-type comparison as a locus-level sanity check, since LuxR counter-silencing by H-NS displacement is expected to concentrate repression relief at quorum sensing loci.

## Related tools

- **Perl** (Hosts the deposited read-processing, counting, and differential expression scripts used to build the count matrix and call H-NS regulon genes at the >2-fold, P < 0.05 threshold) — https://github.com/Juliacvk/LuxR_H-NS
- **getGenePeakAnno2.pl** (Perl helper from the same repository that links regulatory peaks to gene information, useful for annotating regulon genes) — https://github.com/Juliacvk/LuxR_H-NS

## Examples

```
perl getGenePeakAnno2.pl  # run within github.com/Juliacvk/LuxR_H-NS after mapping GSE136191 reads and building the four-strain count matrix
```

## Evaluation signals

- Wild-type versus Δhns comparison yields 726 H-NS-regulated genes, of which 596 are repressed.
- ΔluxR versus ΔluxR Δhns comparison yields 824 H-NS-regulated genes, of which 430 are repressed.
- luxCDABE genes are present in the ΔluxR background regulon, as expected.
- Extracted luxC fold change from the wild-type versus Δhns comparison is consistent with the reported ~1.3-fold change.
- Count matrix contains gene-level counts for all four strains (wild-type, Δhns, ΔluxR, ΔluxR Δhns) at OD600 = 1.0.

## Limitations

- Regulon size is threshold-dependent (genes must pass >2-fold change and P < 0.05), so borderline genes are excluded.
- luxC shows only a modest ~1.3-fold change in the wild-type comparison, indicating locus-level effects can be subtle even when the regulon is large.
- Additional transcriptional and/or post-transcriptional repression mechanisms act at luxCDABE at low cell density even without hns, so the called regulon does not capture all regulatory control.

## Evidence

- [other] Regulon size and threshold: [evidence span withheld — anchor status: not_found]
- [supplementary] Deposited analysis code: "All Perl scripts have been deposited in Github"
- [abstract] Mechanistic rationale: "LuxR derepresses quorum sensing loci by competing with H-NS, a global transcriptional repressor that oligomerizes on DNA to form filaments and bridges"
- [readme] Peak-to-gene annotation helper: "getGenePeakAnno2.pl - perl script to add peaks to gene information"
