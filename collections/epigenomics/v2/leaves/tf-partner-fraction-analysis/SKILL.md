---
name: tf-partner-fraction-analysis
description: Use when you need to annotate a defined TF set with ssDNA-binding domains
  (OB, KH, RRM, sPC4-like clans) and compute the fraction of TFs having at least one
  direct interaction partner that carries such a domain.
when_to_use_negative:
- Your goal is to validate ChIP-seq peaks on HOT regions (e.g. motif or KO signal
  filtering) rather than to annotate TFs with ssDNA-binding domains
- The proteins of interest are not reviewed human Swiss-Prot entries, or you lack
  a curated physical-interaction source comparable to iRefWeb
edam_operation: http://edamontology.org/operation_0306
tools:
- name: HMMER3
  role: hmmsearch scans UniProt Swiss-Prot human sequences against PFAM HMM profiles
    to detect ssDNA-binding domains
- name: PFAM
  role: Provides the HMM profiles for ssDNA-binding domains, restricted to the OB,
    KH, RRM, and sPC4-like clans
- name: UniProt
  role: Source of reviewed (Swiss-Prot) human protein sequences scanned for domains
- name: iRefWeb
  role: Source of human protein-protein interactions, filtered to the paper's physical-interaction
    criteria to find direct partners of the 165 TFs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:ef13d20e847928a76736109d5177ac303ff3f2df2ea525df4c2ce7ffc0c418f6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMMER3
  - PFAM
  - UniProt
  - iRefWeb
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tf-partner-fraction-analysis

## Summary

Quantifies how many transcription factors (TFs) occupying HOT regions can bind single-stranded DNA (ssDNA) directly or via direct interaction partners, combining HMMER3/PFAM domain scanning of Swiss-Prot sequences with physical-interaction filtering of iRefWeb PPI data.

## When to use

Use when you need to annotate a defined TF set with ssDNA-binding domains (OB, KH, RRM, sPC4-like clans) and compute the fraction of TFs having at least one direct interaction partner that carries such a domain.

## When NOT to use

- Your goal is to validate ChIP-seq peaks on HOT regions (e.g. motif or KO signal filtering) rather than to annotate TFs with ssDNA-binding domains
- The proteins of interest are not reviewed human Swiss-Prot entries, or you lack a curated physical-interaction source comparable to iRefWeb

## Inputs

- Reviewed (Swiss-Prot) human protein sequences from UniProt (FASTA)
- PFAM HMM profiles for ssDNA-binding domains (OB, KH, RRM, sPC4-like clans)
- iRefWeb human protein-protein interaction records
- List of the 165 studied TFs occupying HOT regions

## Outputs

- ssdna_partner_fraction.csv with per-TF ssDNA-binding annotations and the summary fraction of TFs with an ssDNA-binding interaction partner

## How to apply

Download reviewed (Swiss-Prot) human protein sequences from UniProt and run HMMER3 hmmsearch against PFAM HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans, collecting proteins with at least one hit. Download iRefWeb human protein-protein interactions and filter to interaction records satisfying the paper's physical-interaction criteria. Join the ssDNA-binding protein list with the filtered iRefWeb edges to determine, for each of the 165 studied TFs, whether at least one direct interaction partner carries an ssDNA-binding domain. Compute the fraction of TFs with such a partner and write per-TF annotations plus the summary fraction to ssdna_partner_fraction.csv.

## Related tools

- **HMMER3** (hmmsearch scans UniProt Swiss-Prot human sequences against PFAM HMM profiles to detect ssDNA-binding domains)
- **PFAM** (Provides the HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans)
- **UniProt** (Source of reviewed (Swiss-Prot) human protein sequences scanned for domains)
- **iRefWeb** (Source of human protein-protein interactions, filtered to the paper's physical-interaction criteria to find direct partners of the 165 TFs)

## Examples

```
hmmsearch --domtblout ssdna_hits.domtblout Pfam_ssDNA_OB_KH_RRM_sPC4.hmm uniprot_sprot_human.fasta
```

## Evaluation signals

- Computed fraction equals 31/165 (18.8%) of TFs with at least one direct interaction partner carrying an ssDNA-binding domain
- Among the TFs themselves, only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) contain ssDNA-binding domains
- ssdna_partner_fraction.csv exists and contains both per-TF annotations and the summary fraction
- Every interaction edge used satisfies the paper's physical-interaction criteria from iRefWeb
- Each domain hit traces to one of the OB, KH, RRM, or sPC4-like PFAM clans

## Limitations

- Direct-partner analysis only covers one-hop physical interactions in iRefWeb; interactions failing the paper's physical-interaction criteria are excluded
- Only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) among the 165 TFs directly carry ssDNA-binding domains, so most TFs depend entirely on partner-based annotation
- Domain detection is limited to the OB, KH, RRM, and sPC4-like PFAM clans; ssDNA binding via other domain families would be missed

## Evidence

- [methods] Run HMMER3 (hmmsearch) against PFAM HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans, and collect proteins with at least one hit.: [evidence span withheld — anchor status: not_found]
- [methods] Download iRefWeb human protein-protein interactions and filter to interaction records satisfying the paper's physical-interaction criteria.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the fraction of TFs with such a partner and write the per-TF annotations and the summary fraction to ssdna_partner_fraction.csv.: [evidence span withheld — anchor status: not_found]
- [abstract] We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA tertiary structures"
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
