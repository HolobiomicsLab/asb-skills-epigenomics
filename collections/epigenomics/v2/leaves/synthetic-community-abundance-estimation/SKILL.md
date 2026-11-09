---
name: synthetic-community-abundance-estimation
description: Use when comparing sequencing methods (RIMS-seq vs DNA-seq vs bisulfite-seq)
  for species-level relative abundance estimation in mixed or synthetic bacterial
  communities with a reference genome per species.
when_to_use_negative:
- The library preparation is based on tagmentation rather than adapter ligation —
  RIMS-seq (and thus its abundance profile) is not valid in that setting
- The goal is precise genomic localization of methylation sites rather than community
  composition — EM-seq or bisulfite-seq are more appropriate for positional methylation
- No reference genome is available and species cannot be distinguished by mapping
  — abundance estimation by per-species read counts requires exact references
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: mapping.pl
  role: Maps sequencing reads to reference genomes with BWA-mem to obtain per-species
    mapped read counts used for abundance normalization
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: BWA
  role: Underlying aligner (bwa mem) invoked by mapping.pl to map reads to the reference
    genome
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:8f29d28dce92db14d2aa32153a0ce787debb569d6135be7a1ff6cec85bccbcc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - mapping.pl
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# synthetic-community-abundance-estimation

## Summary

Estimates normalized per-species relative abundances in ATCC synthetic microbiome communities (MSA-1005/1006) by mapping RIMS-seq, DNA-seq(+3H), and bisulfite-seq reads with BWA-mem and dividing each species' mapped read count by total mapped reads per library, enabling cross-method abundance comparison.

## When to use

Use when comparing sequencing methods (RIMS-seq vs DNA-seq vs bisulfite-seq) for species-level relative abundance estimation in mixed or synthetic bacterial communities with a reference genome per species.

## When NOT to use

- The library preparation is based on tagmentation rather than adapter ligation — RIMS-seq (and thus its abundance profile) is not valid in that setting
- The goal is precise genomic localization of methylation sites rather than community composition — EM-seq or bisulfite-seq are more appropriate for positional methylation
- No reference genome is available and species cannot be distinguished by mapping — abundance estimation by per-species read counts requires exact references

## Inputs

- Per-species mapped read count table for ATCC MSA-1005/1006 synthetic community libraries (RIMS-seq, DNA-seq(+3H), bisulfite-seq)
- BAM files from BWA-mem mapping of paired-end reads to reference genomes (e.g. generated with mapping.pl)
- Reference genomes for each species in the synthetic community

## Outputs

- abundance_comparison.csv with normalized per-species relative abundances and RIMS-seq/DNA-seq and bisulfite-seq/DNA-seq ratio columns
- Per-method relative abundance profiles of the synthetic community

## How to apply

Load the per-species mapped read count table for the ATCC MSA-1005/1006 synthetic gut and skin communities, where libraries (RIMS-seq, DNA-seq, bisulfite-seq) were mapped with BWA-mem via mapping.pl (adapters removed beforehand). Normalize counts to relative abundance per species by dividing each species' mapped read count by the total mapped reads in the same library using pandas. Compute the RIMS-seq/DNA-seq abundance ratio and the bisulfite-seq/DNA-seq abundance ratio for every species. Acceptance criteria: all RIMS-seq/DNA-seq ratios fall within 0.8–1.2, while bisulfite-seq overestimates AT-rich species — Clostridioides difficile at ~2.65x and Staphylococcus epidermidis at ~3.9x relative to DNA-seq, as reported in Figure 5A. Write normalized abundances plus both ratio columns per species to abundance_comparison.csv.

## Related tools

- **mapping.pl** (Maps sequencing reads to reference genomes with BWA-mem to obtain per-species mapped read counts used for abundance normalization) — https://github.com/Ettwiller/RIMS-seq/
- **BWA** (Underlying aligner (bwa mem) invoked by mapping.pl to map reads to the reference genome) — https://github.com/lh3/bwa

## Examples

```
df['rel_abund'] = df['mapped_reads'] / df.groupby('library')['mapped_reads'].transform('sum')
```

## Evaluation signals

- All RIMS-seq/DNA-seq abundance ratios fall within the 0.8–1.2 acceptance window
- Bisulfite-seq/DNA-seq ratio for Clostridioides difficile is approximately 2.65 and for Staphylococcus epidermidis approximately 3.9 (AT-rich overestimation)
- Each species' relative abundance sums across species to 1 within each library (normalization invariant)
- abundance_comparison.csv contains per-species rows with abundance and both ratio columns populated

## Limitations

- Bisulfite-seq biases abundances of AT-rich species (e.g. C. difficile, S. epidermidis), so it is unreliable for abundance estimation
- Uneven species representation in microbiomes can cause RIMS-seq to identify motifs only in the most abundant species
- Closely related species co-existing in a community can make assignment of reads/motifs to the correct species challenging

## Evidence

- [results] Species abundances are similar between DNA-seq(+3H) and RIMS-seq (abundance ratios between 0.8 and 1.2): [evidence span withheld — anchor status: not_found]
- [results] Bisulfite-seq overestimates AT-rich species: [evidence span withheld — anchor status: not_found]
- [methods] mapping.pl maps reads with BWA-mem: "This script uses BWA-mem to map the reads to a reference genome. Adapters need to be removed prior using this script."
- [readme] RIMS-seq protocol resembles DNA-seq: "RIMS-seq (*Rapid Identification of Methylase Specificity*) simultaneously sequences bacterial genomes and determines m5C methylase specificities using a simple experimental protocol"
