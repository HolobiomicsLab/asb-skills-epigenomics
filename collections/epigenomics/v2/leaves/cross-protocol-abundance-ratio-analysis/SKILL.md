---
name: cross-protocol-abundance-ratio-analysis
description: Use when multiple shotgun libraries of the same synthetic or mixed microbial
  community (e.g. RIMS-seq +3H, DNA-seq, bisulfite-seq) have been mapped to the same
  reference and you need to check whether protocol choice distorts species relative
  abundances.
when_to_use_negative:
- The goal is de novo methylase specificity/motif discovery from RIMS-seq reads (use
  the mapping.pl -> split_mapped_reads.pl -> get_motif_all.pl/MoSDi pipeline instead)
- Libraries are not mapped to the same reference genomes with BWA-mem, so per-species
  counts across protocols are not comparable and ratios would be meaningless
- Assessing sequencing quality (coverage, GC content, mapping rate) rather than abundance
  bias between protocols
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mapping.pl
  role: Maps RIMS-seq, DNA-seq and bisulfite-seq reads to the reference genomes with
    BWA-mem to produce the per-species mapped read counts used for abundance normalization
  repo: https://github.com/Ettwiller/RIMS-seq/
- name: BWA
  role: Alignment engine used by mapping.pl (BWA-mem) for read-to-reference mapping
    of all three library types
  repo: https://github.com/lh3/bwa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:0dac4445ce925b3cdbf281820e6ec21bbc9893300dfb8f5b4a11c8cc7b5d5d30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ettwiller/RIMS-seq
  tools:
  - BWA
  - mapping.pl
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Cross-protocol abundance ratio analysis

## Summary

Quantifies per-species relative abundances from BWA-mem-mapped read counts of RIMS-seq, DNA-seq and bisulfite-seq libraries of the ATCC MSA-1005/1006 synthetic microbiomes, and computes pairwise protocol-to-DNA-seq abundance ratios to test whether a sequencing chemistry biases AT-rich species.

## When to use

Use when multiple shotgun libraries of the same synthetic or mixed microbial community (e.g. RIMS-seq +3H, DNA-seq, bisulfite-seq) have been mapped to the same reference and you need to check whether protocol choice distorts species relative abundances.

## When NOT to use

- The goal is de novo methylase specificity/motif discovery from RIMS-seq reads (use the mapping.pl -> split_mapped_reads.pl -> get_motif_all.pl/MoSDi pipeline instead)
- Libraries are not mapped to the same reference genomes with BWA-mem, so per-species counts across protocols are not comparable and ratios would be meaningless
- Assessing sequencing quality (coverage, GC content, mapping rate) rather than abundance bias between protocols

## Inputs

- Per-species mapped read count table for ATCC MSA-1005/1006 synthetic gut and skin communities (RIMS-seq, DNA-seq and bisulfite-seq libraries mapped with BWA-mem via mapping.pl)
- List of species/community members with per-library mapped read totals

## Outputs

- abundance_comparison.csv containing normalized relative abundances per species per library, RIMS-seq/DNA-seq abundance ratios, and bisulfite-seq/DNA-seq abundance ratios

## How to apply

Load the per-species mapped read count table (produced with mapping.pl, which uses BWA-mem to map reads to the reference genomes of the ATCC MSA-1005/1006 synthetic gut and skin communities) in pandas. Normalize each species' mapped read count by the total mapped reads within the same library to obtain relative abundance per protocol. Compute the RIMS-seq/DNA-seq ratio and the bisulfite-seq/DNA-seq ratio for every species. Acceptance criteria from the paper (Figure 5A): all RIMS-seq/DNA-seq ratios fall within 0.8–1.2, whereas bisulfite-seq overestimates AT-rich species, with Clostridioides difficile at approximately 2.65x and Staphylococcus epidermidis at approximately 3.9x relative to DNA-seq. Write the normalized abundances and both ratio columns per species to abundance_comparison.csv.

## Related tools

- **mapping.pl** (Maps RIMS-seq, DNA-seq and bisulfite-seq reads to the reference genomes with BWA-mem to produce the per-species mapped read counts used for abundance normalization) — https://github.com/Ettwiller/RIMS-seq/
- **BWA** (Alignment engine used by mapping.pl (BWA-mem) for read-to-reference mapping of all three library types) — https://github.com/lh3/bwa

## Examples

```
df['ab'] = df.groupby('library')['mapped_reads'].transform(lambda x: x/x.sum()); df['rims_vs_dna'] = df['ab_rims']/df['ab_dna']; df.to_csv('abundance_comparison.csv')
```

## Evaluation signals

- All RIMS-seq/DNA-seq per-species abundance ratios fall within the 0.8–1.2 window reported in Figure 5A
- Bisulfite-seq/DNA-seq ratios for AT-rich species reproduce the reported overestimates: C. difficile ≈ 2.65x and S. epidermidis ≈ 3.9x
- Per-library relative abundances sum to 1.0 after normalization by total mapped reads
- abundance_comparison.csv contains one row per community species with both ratio columns populated
- RIMS-seq QC metrics (coverage, GC content, mapping rate) are consistent with DNA-seq, supporting comparability of the mapped counts

## Limitations

- In natural, unevenly represented microbiomes, RIMS-seq may identify motifs only in the most abundant species, complicating cross-species comparisons
- Closely related species co-existing in communities make assigning motifs (and, by extension, species-resolved counts) to the correct species challenging
- Single nucleotide polymorphisms in microbial communities may confound C-to-T deamination detection, increasing background noise in motif-based analyses

## Evidence

- [results] Species abundances are similar between DNA-seq(+3H) and RIMS-seq (abundance ratios between 0.8 and 1.2): [evidence span withheld — anchor status: not_found]
- [methods] This script uses BWA-mem to map the reads to a reference genome.: "This script uses BWA-mem to map the reads to a reference genome."
- [discussion] Sequencing QC metrics such as coverage, GC content and mapping rate are similar for RIMS-seq and DNA-seq.: "Sequencing QC metrics such as coverage, GC content and mapping rate are similar for RIMS-seq and DNA-seq."
