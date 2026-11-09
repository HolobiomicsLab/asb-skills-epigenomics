---
name: tissue-specific-gene-set-annotation
description: Use when you have a table of recurrent chromatin state patterns from
  EpiAlign horizontal alignment and need to determine which patterns carry cell-type-specific
  characteristics via tissue-specific gene overlap.
when_to_use_negative:
- Motifs you are analyzing still contain the quiescent state 'o', which the authors
  exclude because it lacks a good biological interpretation.
- You need cross-sample similarity rather than within-epigenome recurrent patterns
  — vertical (cross-epigenome) alignment or clustering tasks need a different skill.
- Your epigenome annotation was not produced by a chromatin state method like ChromHMM;
  EpiAlign requires single-track chromatin state sequences as input.
tools:
- name: EpiAlign
  role: Performs the Smith-Waterman-style local alignment of 500 kb query regions
    whose best-matching pairs (e.g. NRG3 vs GRIA1 in E071) supply the recurrent motifs
    being annotated.
  repo: https://github.com/zzz3639/EpiAlign
- name: ChromHMM
  role: Encodes multi-track epigenomic signals into the single-track chromatin state
    sequences whose recurrent patterns are annotated with gene sets.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:d7bb25dcd9bf39aca484c3cc4a69787eb55472a469dca96f0180e876dfe447c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# tissue-specific-gene-set-annotation

## Summary

Annotates EpiAlign horizontal-alignment pattern tables (recurrent chromatin state motifs extracted from a single Roadmap epigenome, e.g. E071 brain) with tissue-specific gene sets and Roadmap sample metadata, so that each motif carries a tissue/cell-type association.

## When to use

Use when you have a table of recurrent chromatin state patterns from EpiAlign horizontal alignment and need to determine which patterns carry cell-type-specific characteristics via tissue-specific gene overlap.

## When NOT to use

- Motifs you are analyzing still contain the quiescent state 'o', which the authors exclude because it lacks a good biological interpretation.
- You need cross-sample similarity rather than within-epigenome recurrent patterns — vertical (cross-epigenome) alignment or clustering tasks need a different skill.
- Your epigenome annotation was not produced by a chromatin state method like ChromHMM; EpiAlign requires single-track chromatin state sequences as input.

## Inputs

- ChromHMM chromatin state sequence for a target Roadmap epigenome (e.g. E071 brain sample)
- EpiAlign horizontal-alignment pattern table of recurrent chromatin state motifs (quiescent-state 'o' motifs removed)
- Tissue-specific gene sets (e.g. brain-specific genes such as NRG3 and GRIA1)
- Roadmap sample metadata (sample IDs and tissue/cell-type labels)
- GENCODE v10 protein-coding gene annotations compatible with the Roadmap database

## Outputs

- epialign_horizontal_patterns.tsv — pattern table annotated with tissue/cell-type associations and overlap with tissue-specific genes

## How to apply

Run EpiAlign's horizontal alignment on a ChromHMM-derived epigenome (500 kb regions, query aligned against all other regions with mismatch penalty epsilon_N = 1.5, deletion penalty epsilon_D = 1, frequency-based weights), collect recurrent aligned state patterns, and exclude any motifs containing the quiescent state 'o' since it 'lacks a good biological interpretation'. Annotate the pattern table with tissue-specific gene sets (e.g. brain-specific genes such as NRG3 and GRIA1 for epigenome E071) using protein-coding gene definitions from GENCODE v10 compatible with the Roadmap database, plus Roadmap sample metadata to assign each pattern a tissue/cell-type association. Quantify how often each pattern overlaps tissue-specific genes (e.g. verifying that NRG3's best alignment is GRIA1 in E071) and write the annotated table to epialign_horizontal_patterns.tsv.

## Related tools

- **EpiAlign** (Performs the Smith-Waterman-style local alignment of 500 kb query regions whose best-matching pairs (e.g. NRG3 vs GRIA1 in E071) supply the recurrent motifs being annotated.) — https://github.com/zzz3639/EpiAlign
- **ChromHMM** (Encodes multi-track epigenomic signals into the single-track chromatin state sequences whose recurrent patterns are annotated with gene sets.)

## Examples

```
git clone https://github.com/zzz3639/EpiAlign
```

## Evaluation signals

- epialign_horizontal_patterns.tsv exists and contains recurrent chromatin state patterns with tissue/cell-type association columns populated.
- Known tissue-specific gene pairs are recovered: in E071, NRG3's best alignment is another brain-specific gene, GRIA1.
- No pattern in the annotated table contains the quiescent state 'o'.
- Pattern-tissue assignments are consistent across samples of the same tissue (hierarchical clustering of horizontal alignment score vectors grouped same-tissue samples together, Figure 5).
- Gene sets were derived from all protein-coding genes in GENCODE v10 compatible with the Roadmap database.

## Limitations

- Motifs containing the quiescent state 'o' are excluded because that state 'lacks a good biological interpretation', so quiescence-dominated patterns cannot be annotated.
- Genes on chromosome Y are excluded from the gene-level analysis on the Roadmap data.
- Results depend on ChromHMM annotations; the discussion notes that 'likely noisy ChromHMM annotations that need further biological validation' can arise (Supplementary section 12).

## Evidence

- [methods] NRG3/GRIA1 best-alignment validation in brain sample: "in the brain sample E071, when we apply EpiAlign with the compression step, the brain-specific gene NRG3 has the best alignment with another brain-specific gene GRIA1"
- [results] quiescent-state filter rationale: [evidence span withheld — anchor status: not_found]
- [results] gene universe definition (GENCODE v10): "we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database"
- [abstract] cell-type-specific patterns finding: "EpiAlign is able to extract recurrent chromatin state patterns along a single epigenome, and many of these patterns carry cell-type-specific characteristics."
- [readme] install via git clone (README): "we recommend you to download our package by "git clone" ruther than downloading the zip file."
