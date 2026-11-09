---
name: protein-interaction-network-filtering
description: Use when you need to annotate a fixed list of proteins (e.g. 165 human
  TFs) with a domain-based capability (OB, KH, RRM, sPC4-like ssDNA-binding) propagated
  one hop through direct physical PPIs, and report the fraction annotated.
when_to_use_negative:
- Your question requires indirect (multi-hop) partners or functional associations
  — this skill propagates annotations only through direct physical interactions.
- You need ChIP-seq peak-level evidence rather than protein-level domain/interaction
  annotations; this skill does not process genomic intervals.
- Your protein set is non-human or includes isoforms not covered by reviewed Swiss-Prot
  entries, since the pipeline is built on Swiss-Prot human sequences.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0080
tools:
- name: HMMER3
  role: hmmsearch of UniProt Swiss-Prot human proteins against PFAM ssDNA-binding
    HMM profiles
- name: PFAM
  role: source of HMM profiles for OB, KH, RRM, and sPC4-like ssDNA-binding domain
    clans
- name: UniProt
  role: provides reviewed (Swiss-Prot) human protein sequences scanned for ssDNA-binding
    domains
- name: iRefWeb
  role: source of human protein-protein interactions filtered to physical-interaction
    records for one-hop partner lookup
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz460
    title: 'HOT or not: examining the basis of high-occupancy target regions'
  source_content_hash: sha256:855e8281c762d7a6323c442ca630316be748ef8c54efb0a7c77f223882438b88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HMMER3
  - PFAM
  - UniProt
  - iRefWeb
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# protein-interaction-network-filtering

## Summary

Identifies whether each of a set of studied transcription factors (e.g. the 165 HOT-region TFs) carries an ssDNA-binding domain itself or has at least one direct physical interaction partner that does, by intersecting HMMER3/PFAM domain hits against a filtered iRefWeb human protein-protein interaction network.

## When to use

Use when you need to annotate a fixed list of proteins (e.g. 165 human TFs) with a domain-based capability (OB, KH, RRM, sPC4-like ssDNA-binding) propagated one hop through direct physical PPIs, and report the fraction annotated.

## When NOT to use

- Your question requires indirect (multi-hop) partners or functional associations — this skill propagates annotations only through direct physical interactions.
- You need ChIP-seq peak-level evidence rather than protein-level domain/interaction annotations; this skill does not process genomic intervals.
- Your protein set is non-human or includes isoforms not covered by reviewed Swiss-Prot entries, since the pipeline is built on Swiss-Prot human sequences.

## Inputs

- Reviewed (Swiss-Prot) human protein sequences from UniProt
- PFAM HMM profiles for ssDNA-binding domains (OB, KH, RRM, sPC4-like clans)
- iRefWeb human protein-protein interaction records
- List of studied transcription factors (the 165 HOT-region TFs)

## Outputs

- ssdna_partner_fraction.csv with per-TF ssDNA-binding domain annotations (own domain and/or direct partner domain)
- Summary fraction of TFs with at least one ssDNA-binding direct partner (31/165 = 18.8%)
- List of proteins with at least one ssDNA-binding HMMER3/PFAM domain hit

## How to apply

Download reviewed (Swiss-Prot) human protein sequences from UniProt and run HMMER3 hmmsearch against PFAM HMM profiles for ssDNA-binding domains, restricting to the OB, KH, RRM, and sPC4-like clans; retain proteins with at least one hit. Download iRefWeb human protein-protein interactions and filter to interaction records satisfying the paper's physical-interaction criteria. For each of the studied TFs (here 165), join the ssDNA-binding protein list with the filtered iRefWeb edges to test whether the TF itself or at least one direct interaction partner carries an ssDNA-binding domain. Compute the per-TF annotations and the summary fraction (expected: 31/165, 18.8%, with partners), validating that only BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) carry domains among the TFs themselves; write results to ssdna_partner_fraction.csv.

## Related tools

- **HMMER3** (hmmsearch of UniProt Swiss-Prot human proteins against PFAM ssDNA-binding HMM profiles)
- **PFAM** (source of HMM profiles for OB, KH, RRM, and sPC4-like ssDNA-binding domain clans)
- **UniProt** (provides reviewed (Swiss-Prot) human protein sequences scanned for ssDNA-binding domains)
- **iRefWeb** (source of human protein-protein interactions filtered to physical-interaction records for one-hop partner lookup)

## Examples

```
hmmsearch --domtblout ssdna_hits.domtbl pfam_ssDNA.hmm uniprot_sprot_human.fasta; python join_iRefWeb_partners.py --tf-list tf_165.csv --hits ssdna_hits.domtbl --irefweb irefweb_human.tsv --out ssdna_partner_fraction.csv
```

## Evaluation signals

- The computed fraction of TFs with at least one ssDNA-binding direct partner equals 31/165 (18.8%).
- BACH1 (DUF1866) and E2F6 (BRCA-2_OB3) are the only TFs among the 165 with ssDNA-binding domains of their own.
- Every protein in the ssDNA-binding list has at least one hmmsearch hit restricted to the OB, KH, RRM, or sPC4-like clans.
- All iRefWeb edges used satisfy the paper's physical-interaction criteria before the join.
- Per-TF output file ssdna_partner_fraction.csv contains an annotation (yes/no for own domain and partner domain) for each of the 165 TFs.

## Limitations

- Only direct interaction partners are considered; indirect propagation through the PPI network is out of scope.
- It is not possible to estimate the proportion of antibody-specific error (e.g. pull-down of non-target proteins vs. direct binding to polynucleotide epitopes) from currently available data, which limits interpretation of TF-HOT region occupancy.
- A lack of signal in a KO ChIP-seq experiment might also be caused by technical conditions such as low number of reads, low library complexity or unsuccessful IP, so TF presence/absence at HOT regions is itself uncertain.

## Evidence

- [methods] hmmsearch against PFAM clans: [evidence span withheld — anchor status: not_found]
- [methods] iRefWeb physical filtering: [evidence span withheld — anchor status: not_found]
- [results] partner fraction finding: [evidence span withheld — anchor status: not_found]
- [readme] repository scripts: "The scripts in this repository have been used for analysis of HOT regions (Scripts/HOT-region-main.R)."
- [intro] motifless binding hypothesis: "the plausible explanations for motifless binding are a combination of 1) interaction of transcriptions factors (TFs) where only a handful of them are actually binding to DNA"
