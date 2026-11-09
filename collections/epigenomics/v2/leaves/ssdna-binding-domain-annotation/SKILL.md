---
name: ssdna-binding-domain-annotation
description: Use when you must determine whether transcription factors (or their direct
  PPI partners) can bind ssDNA, e.g. to link HOT-region TFs to R-loop or G-quadruplex-associated
  sequence features.
when_to_use_negative:
- When the question is genome-wide motif occupancy at HOT regions rather than protein
  domain annotation — the article shows HOT regions lack canonical TF motifs and this
  skill addresses protein-level ssDNA binding, not DNA motif discovery.
- When interaction data cannot satisfy physical-interaction criteria (e.g. genetic
  or predicted-only PPI edges), since partner-level percentages would be inflated
  by indirect associations.
- When working with non-reviewed/unreviewed UniProt entries, since domain calls were
  calibrated against the Swiss-Prot human proteome.
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: HMMER3
  role: hmmsearch scans human protein sequences against PFAM ssDNA-binding HMM profiles
    to detect OB, KH, RRM, and sPC4-like clan domains
- name: PFAM
  role: source of curated HMM profiles for ssDNA-binding domain families
- name: UniProt
  role: provides the reviewed (Swiss-Prot) human protein sequence input
- name: iRefWeb
  role: source of human protein-protein interaction edges filtered to physical interactions
    for the partner-level annotation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:397c48f214e7ee7e8557d218c8a8e2500b67244e1b85c2754f0995f481bc5633
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMMER3
  - PFAM
  - UniProt
  - iRefWeb
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# ssDNA-binding-domain-annotation

## Summary

Annotates human proteins as single-stranded DNA (ssDNA)-binding via HMMER3 profile searches against PFAM HMMs of the OB, KH, RRM, and sPC4-like clans, and extends the annotation to direct interaction partners using iRefWeb physical PPI edges to compute the fraction of transcription factors with at least one ssDNA-binding partner.

## When to use

Use when you must determine whether transcription factors (or their direct PPI partners) can bind ssDNA, e.g. to link HOT-region TFs to R-loop or G-quadruplex-associated sequence features.

## When NOT to use

- When the question is genome-wide motif occupancy at HOT regions rather than protein domain annotation — the article shows HOT regions lack canonical TF motifs and this skill addresses protein-level ssDNA binding, not DNA motif discovery.
- When interaction data cannot satisfy physical-interaction criteria (e.g. genetic or predicted-only PPI edges), since partner-level percentages would be inflated by indirect associations.
- When working with non-reviewed/unreviewed UniProt entries, since domain calls were calibrated against the Swiss-Prot human proteome.

## Inputs

- Reviewed (Swiss-Prot) human protein sequences from UniProt
- PFAM HMM profiles restricted to the OB, KH, RRM, and sPC4-like clans (ssDNA-binding domains)
- iRefWeb human protein-protein interaction records (physical-interaction criteria applied)
- List of the 165 transcription factors studied for HOT-region occupancy

## Outputs

- ssdna_partner_fraction.csv containing per-TF ssDNA-binding annotations and the summary fraction of TFs with an ssDNA-binding direct partner
- List of human proteins with at least one ssDNA-binding domain hit from hmmsearch
- Binary per-TF flag: has at least one direct interaction partner with an ssDNA-binding domain

## How to apply

Download reviewed (Swiss-Prot) human protein sequences from UniProt and run HMMER3 hmmsearch against PFAM HMM profiles for ssDNA-binding domains, restricting hits to the OB, KH, RRM, and sPC4-like clans, collecting every protein with at least one domain hit. In parallel, download iRefWeb human protein-protein interactions and filter to interaction records that satisfy physical-interaction criteria. Join the ssDNA-binding protein list with the filtered iRefWeb edges to decide, for each of the 165 studied TFs, whether at least one direct interaction partner carries an ssDNA-binding domain. Compute the per-TF annotations and the summary fraction, and validate against the published result of 31/165 TFs (18.8%) with an ssDNA-binding partner, with BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) as the only TFs carrying domains themselves.

## Related tools

- **HMMER3** (hmmsearch scans human protein sequences against PFAM ssDNA-binding HMM profiles to detect OB, KH, RRM, and sPC4-like clan domains)
- **PFAM** (source of curated HMM profiles for ssDNA-binding domain families)
- **UniProt** (provides the reviewed (Swiss-Prot) human protein sequence input)
- **iRefWeb** (source of human protein-protein interaction edges filtered to physical interactions for the partner-level annotation)

## Examples

```
hmmsearch --tblout ssdna_hits.tbl pfam_ssdna_OB_KH_RRM_sPC4.hmm uniprot_sprot_human.fasta
```

## Evaluation signals

- The computed fraction of TFs with at least one ssDNA-binding direct interaction partner equals 31/165 (18.8%).
- Among the 165 TFs themselves, only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) contain ssDNA-binding domains.
- Every reported domain hit belongs to one of the OB, KH, RRM, or sPC4-like PFAM clans.
- All interaction edges used pass the paper's physical-interaction criteria applied to iRefWeb records.
- ssdna_partner_fraction.csv contains one annotation row per studied TF plus the summary fraction.

## Limitations

- HOT regions lack canonical motifs for the TFs thought to be bound there, and with current data the proportion of antibody-specific error from pull-down of non-target proteins vs. direct binding to polynucleotide epitopes cannot be estimated, so domain- and partner-based explanations for motifless binding remain partial.
- Hypothesized explanations for motifless binding are a combination of TF-TF interactions, weak binding to non-canonical motifs, and hyper-ChIPable regions — this skill only captures the first (interaction-partner) mechanism.
- Partner-level conclusions depend on the completeness and physical-interaction filtering of iRefWeb; missing or spurious edges directly change the 18.8% figure.

## Evidence

- [methods] Run HMMER3 (hmmsearch) against PFAM HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans, and collect proteins with at least one hit.: [evidence span withheld — anchor status: not_found]
- [results] 31 out of 165 TFs (18.8%) have at least one direct interaction partner with an ssDNA-binding domain; among the TFs themselves, only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) contain ssDNA-binding domains.: [evidence span withheld — anchor status: not_found]
- [abstract] HOT regions have discriminatory sequence characteristics and genomic features including GC/CpG-rich k-mers, R-loops, and G-quadruplex DNA: "We observe sequence characteristics and genomic features that are discriminatory of HOT regions, such as GC/CpG-rich k-mers, enrichment of RNA-DNA hybrids (R-loops) and DNA"
- [readme] The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R).: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
