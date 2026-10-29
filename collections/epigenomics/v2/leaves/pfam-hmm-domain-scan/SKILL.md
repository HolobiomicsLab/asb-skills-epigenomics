---
name: pfam-hmm-domain-scan
description: Use when a research question asks whether a set of transcription factors
  (e.g. TFs occupying HOT regions) can bind single-stranded DNA directly or through
  their direct interaction partners, requiring domain-level annotation via PFAM HMMs
  plus PPI evidence.
when_to_use_negative:
- When the question is about motif occupancy or ChIP-seq peak enrichment on HOT regions
  rather than protein domain content and PPI-mediated ssDNA binding capability
- When only non-reviewed (TrEMBL) or non-human proteomes are available, since the
  workflow is calibrated to Swiss-Prot human sequences and iRefWeb human interactions
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0625
tools:
- name: HMMER3
  role: hmmsearch engine used to scan human Swiss-Prot sequences against PFAM ssDNA-binding
    HMM profiles
- name: PFAM
  role: Source of HMM profiles for ssDNA-binding domains in the OB, KH, RRM, and sPC4-like
    clans
- name: UniProt
  role: Provides the reviewed (Swiss-Prot) human protein sequences used as the hmmsearch
    target database
- name: iRefWeb
  role: Source of human protein-protein interactions, filtered to physical-interaction
    criteria, to identify direct partners of the 165 TFs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:3eafd454bdbf649c97392f5de85509c60e25e2f62b3280a2ad07e7c1bac92430
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMMER3
  - PFAM
  - UniProt
  - iRefWeb
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pfam-hmm-domain-scan

## Summary

Scan reviewed human protein sequences with HMMER3 (hmmsearch) against PFAM HMM profiles of ssDNA-binding domains (OB, KH, RRM, and sPC4-like clans) to determine which proteins carry ssDNA-binding domains, then join the hit list with iRefWeb physical protein-protein interactions to ask whether transcription factors bind ssDNA directly or via direct partners.

## When to use

Use when a research question asks whether a set of transcription factors (e.g. TFs occupying HOT regions) can bind single-stranded DNA directly or through their direct interaction partners, requiring domain-level annotation via PFAM HMMs plus PPI evidence.

## When NOT to use

- When the question is about motif occupancy or ChIP-seq peak enrichment on HOT regions rather than protein domain content and PPI-mediated ssDNA binding capability
- When only non-reviewed (TrEMBL) or non-human proteomes are available, since the workflow is calibrated to Swiss-Prot human sequences and iRefWeb human interactions

## Inputs

- Reviewed (Swiss-Prot) human protein sequences from UniProt (FASTA)
- PFAM HMM profiles for ssDNA-binding domains from the OB, KH, RRM, and sPC4-like clans
- iRefWeb human protein-protein interaction records
- List of studied transcription factors (165 TFs occupying HOT regions)

## Outputs

- ssdna_partner_fraction.csv with per-TF ssDNA-domain and partner annotations plus the summary fraction
- List of proteins with at least one HMMER3 hit to an ssDNA-binding PFAM HMM
- Summary fraction of TFs with at least one ssDNA-binding direct interaction partner (31/165, 18.8%)

## How to apply

Download reviewed (Swiss-Prot) human protein sequences from UniProt as the search space. Run HMMER3 (hmmsearch) against PFAM HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans, and collect proteins with at least one hit. Download iRefWeb human protein-protein interactions and filter to interaction records satisfying the paper's physical-interaction criteria. Join the ssDNA-binding protein list with the filtered iRefWeb edges so that, for each studied TF, you test whether at least one direct interaction partner carries an ssDNA-binding domain; compute the per-TF annotations and the summary fraction (31/165 TFs, 18.8%, in the source study) and write them to ssdna_partner_fraction.csv.

## Related tools

- **HMMER3** (hmmsearch engine used to scan human Swiss-Prot sequences against PFAM ssDNA-binding HMM profiles)
- **PFAM** (Source of HMM profiles for ssDNA-binding domains in the OB, KH, RRM, and sPC4-like clans)
- **UniProt** (Provides the reviewed (Swiss-Prot) human protein sequences used as the hmmsearch target database)
- **iRefWeb** (Source of human protein-protein interactions, filtered to physical-interaction criteria, to identify direct partners of the 165 TFs)

## Examples

```
hmmsearch --domtblout ssdna_hits.domtbl pfam_ssdna_OB_KH_RRM_sPC4.hmm uniprot_swissprot_human.fasta
```

## Evaluation signals

- The computed fraction of TFs with at least one ssDNA-binding direct interaction partner equals 31/165 (18.8%)
- Among the TFs themselves, BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) are the only ones with ssDNA-binding domains
- Every reported HMMER3 hit maps to a PFAM clan within the allowed set (OB, KH, RRM, sPC4-like)
- Only iRefWeb interaction records passing the paper's physical-interaction criteria contribute to the partner-join
- ssdna_partner_fraction.csv contains a per-TF annotation row for each of the 165 studied TFs plus the summary fraction

## Limitations

- Only two of the 165 TFs themselves carry ssDNA-binding domains (BACH1 via DUF1866, E2F6 via BRCA-2_OB3), so most of the signal comes from interaction partners, making the result sensitive to iRefWeb completeness and physical-interaction filtering
- Domain detection is restricted to OB, KH, RRM, and sPC4-like clans; ssDNA-binding capability outside these clans will be missed
- Whether an ssDNA-binding interaction partner connects the TF to ssDNA mechanistically depends on the physical-interaction criteria used; in the article context it was not possible to disentangle such indirect effects (e.g. the discussion notes that estimating the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding was not possible with available data)

## Evidence

- [methods] Run HMMER3 (hmmsearch) against PFAM HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans: [evidence span withheld — anchor status: not_found]
- [methods] iRefWeb join and fraction computation: [evidence span withheld — anchor status: not_found]
- [results] 31/165 partner fraction finding: [evidence span withheld — anchor status: not_found]
- [abstract] Motivation: HOT regions show ssDNA-related features: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops)"
- [readme] Repository grounding: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
