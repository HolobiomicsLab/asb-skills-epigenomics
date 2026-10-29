---
name: irefweb-interaction-curation
description: Use when assessing whether proteins of interest (e.g. a set of 165 transcription
  factors occupying HOT regions) can perform a function directly or through direct
  physical interaction partners, using iRefWeb interaction data joined with a protein-domain
  scan.
when_to_use_negative:
- When the question is about indirect or pathway-level associations rather than direct
  physical interaction partners
- When the proteins of interest are not covered by iRefWeb human interaction records
  or by reviewed UniProt (Swiss-Prot) sequences
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0602
tools:
- name: iRefWeb
  role: Source of human protein-protein interaction records, filtered to physical-interaction
    criteria before joining with the ssDNA-binding protein list
- name: HMMER3
  role: Runs hmmsearch of human protein sequences against PFAM HMM profiles to identify
    proteins with ssDNA-binding domains
- name: PFAM
  role: Provides HMM profiles for ssDNA-binding domain clans (OB, KH, RRM, sPC4-like)
    used as search targets
- name: UniProt
  role: Supplies reviewed (Swiss-Prot) human protein sequences as the search database
    for the domain scan
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:80a65f233bc2690bd815907632297a07dbb7223600b87b2f269cf6ed4fe8d2c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMMER3
  - PFAM
  - UniProt
  - iRefWeb
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# irefweb-interaction-curation

## Summary

Curate human protein-protein interactions from iRefWeb by filtering to records that satisfy physical-interaction criteria, then join the resulting edges to a domain-scan protein list to test whether each transcription factor (TF) has a direct partner carrying an ssDNA-binding domain.

## When to use

Use when assessing whether proteins of interest (e.g. a set of 165 transcription factors occupying HOT regions) can perform a function directly or through direct physical interaction partners, using iRefWeb interaction data joined with a protein-domain scan.

## When NOT to use

- When the question is about indirect or pathway-level associations rather than direct physical interaction partners
- When the proteins of interest are not covered by iRefWeb human interaction records or by reviewed UniProt (Swiss-Prot) sequences

## Inputs

- Reviewed (Swiss-Prot) human protein sequences from UniProt
- PFAM HMM profiles for ssDNA-binding domains (OB, KH, RRM, sPC4-like clans)
- iRefWeb human protein-protein interaction records
- List of 165 studied transcription factors

## Outputs

- Per-TF annotation of whether at least one direct interaction partner carries an ssDNA-binding domain
- Summary fraction of TFs with an ssDNA-binding interaction partner (ssdna_partner_fraction.csv)

## How to apply

Download iRefWeb human protein-protein interaction records and filter them so only interaction records satisfying the paper's physical-interaction criteria are retained. Separately, build a list of ssDNA-binding proteins by running HMMER3 hmmsearch on reviewed (Swiss-Prot) human sequences from UniProt against PFAM HMM profiles restricted to the OB, KH, RRM, and sPC4-like clans, keeping proteins with at least one hit. Join the ssDNA-binding protein list with the filtered iRefWeb edges: for each of the 165 studied TFs, determine whether at least one direct interaction partner carries an ssDNA-binding domain. Compute the fraction of TFs with such a partner and write per-TF annotations plus the summary fraction to ssdna_partner_fraction.csv; validate against the reported 31/165 (18.8%) and the fact that only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) themselves contain ssDNA-binding domains.

## Related tools

- **iRefWeb** (Source of human protein-protein interaction records, filtered to physical-interaction criteria before joining with the ssDNA-binding protein list)
- **HMMER3** (Runs hmmsearch of human protein sequences against PFAM HMM profiles to identify proteins with ssDNA-binding domains)
- **PFAM** (Provides HMM profiles for ssDNA-binding domain clans (OB, KH, RRM, sPC4-like) used as search targets)
- **UniProt** (Supplies reviewed (Swiss-Prot) human protein sequences as the search database for the domain scan)

## Examples

```
hmmsearch --domtblout ssdna_hits.domtbl pfam_ssDNA.hmm human_swissprot.fasta; python scripts/join_partners.py --interactions irefweb_human.tsv --domains ssdna_hits.domtbl --tfs tf_list.txt --out ssdna_partner_fraction.csv
```

## Evaluation signals

- The computed fraction of TFs with at least one ssDNA-binding direct partner equals 31/165 (18.8%)
- Only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) appear as TFs with ssDNA-binding domains in the direct-domain scan
- Every reported partner is connected by an iRefWeb record that satisfies the paper's physical-interaction criteria
- ssdna_partner_fraction.csv contains both per-TF annotations and the summary fraction with consistent counts

## Limitations

- Only direct interaction partners are considered, so functional ssDNA handling via indirect interactors is missed
- The analysis depends on iRefWeb coverage and the physical-interaction filter; interaction records failing these criteria are excluded
- Domain detection is restricted to OB, KH, RRM, and sPC4-like PFAM clans, so ssDNA-binding domains outside these clans are not counted

## Evidence

- [other] 31 out of 165 TFs (18.8%) have at least one direct interaction partner with an ssDNA-binding domain: [evidence span withheld — anchor status: not_found]
- [other] Run HMMER3 (hmmsearch) against PFAM HMM profiles for ssDNA-binding domains, restricted to the OB, KH, RRM, and sPC4-like clans: [evidence span withheld — anchor status: not_found]
- [intro] the plausible explanations for motifless binding are a combination of 1) interaction of transcriptions factors (TFs): "the plausible explanations for motifless binding are a combination of 1) interaction of transcriptions factors (TFs) where only a handful of them are actually binding to DNA"
- [readme] The scripts in this repository have been used for analysis of HOT regions: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
