---
name: chromatin-state-annotation
description: Use when you have genome-wide multi-track epigenomic signals (histone
  mark ChIP-seq tracks) for one or more Roadmap/ENCODE samples and need discrete chromatin
  state sequences as input to EpiAlign alignment or compression benchmarking.
when_to_use_negative:
- Input is already a single-track chromatin state sequence or a compressed state-label
  sequence — re-annotating with ChromHMM is unnecessary; proceed directly to EpiAlign
  alignment
- The goal is comparing raw multi-track signal matrices rather than chromatin state
  sequences — EpiAlign's premise is to reduce signal comparison to state sequence
  comparison, not to align signal matrices directly
- Regions of interest are dominated by the quiescent state 'o', which lacks a good
  biological interpretation and is excluded from motif analysis in the article
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: ChromHMM
  role: Hidden Markov model that assigns discrete chromatin state labels to genomic
    regions based on signals of multiple epigenetic marks, producing the single-track
    state sequences used by EpiAlign
- name: EpiAlign
  role: 'Downstream consumer: performs local alignment (modified Smith–Waterman dynamic
    programming with state weights, εN=1.5, εD=1) on the annotated (and compressed)
    chromatin state sequences'
  repo: https://github.com/zzz3639/EpiAlign
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkz287
    title: 'EpiAlign: an alignment-based bioinformatic tool for comparing chromatin
      state sequences'
  source_content_hash: sha256:8eb6efd7752526beb52a5fd4b2f44f727cf7ec80535743e0d8988402d74ab59c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zzz3639/EpiAlign
  tools:
  - ChromHMM
  - EpiAlign
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chromatin-state-annotation

## Summary

Annotates genome-wide multi-track epigenomic signals (e.g. ChIP-seq histone marks from NIH Roadmap samples such as E071 brain) into single-track chromatin state sequences using a hidden Markov model method such as ChromHMM, producing the discrete state-label sequences that EpiAlign consumes for local alignment.

## When to use

Use when you have genome-wide multi-track epigenomic signals (histone mark ChIP-seq tracks) for one or more Roadmap/ENCODE samples and need discrete chromatin state sequences as input to EpiAlign alignment or compression benchmarking.

## When NOT to use

- Input is already a single-track chromatin state sequence or a compressed state-label sequence — re-annotating with ChromHMM is unnecessary; proceed directly to EpiAlign alignment
- The goal is comparing raw multi-track signal matrices rather than chromatin state sequences — EpiAlign's premise is to reduce signal comparison to state sequence comparison, not to align signal matrices directly
- Regions of interest are dominated by the quiescent state 'o', which lacks a good biological interpretation and is excluded from motif analysis in the article

## Inputs

- Genome-wide multi-track epigenomic signal tracks (e.g. histone mark ChIP-seq bigWig/bedGraph files) from NIH Roadmap samples such as E071 (brain) or E003 (ESC)
- ChromHMM-trained state model (e.g. 7-state ChromHMM model as in Yue et al. 2014 mouse-human joint training)
- Gene coordinates from GENCODE v10 (protein-coding genes compatible with the Roadmap database, excluding chromosome Y) for gene-region annotations

## Outputs

- Single-track chromatin state sequences per sample per genomic region (chromosome, 500 kb region, or gene)
- Compressed state sequences in which consecutive occurrences of the same state are collapsed into one state label
- Per-window discrete state labels (e.g. 25 bp resolution) ready for EpiAlign local alignment scoring

## How to apply

Apply a chromatin state annotation method — ChromHMM in this article — to encode multi-track epigenomic signals (e.g. the 2804 genome-wide datasets of NIH Roadmap release 9 across 127 tissue/cell types) into single-track chromatin state sequences, where each 25 bp genomic window receives a discrete state label (e.g. the 7-state model used with the Yue et al. mouse-human data). The resulting per-sample, per-region state sequences (whole chromosomes, 500 kb regions, or gene regions such as the 19,935 GENCODE v10 protein-coding genes compatible with Roadmap) then feed EpiAlign's modified Smith–Waterman local alignment (mismatch penalty εN=1.5, deletion penalty εD=1). Before alignment, compress consecutive occurrences of the same chromatin state into a single state label, since uncompressed sequences contain long stretches of a single state (notably the quiescent state 'o') that dominate run time (~20x slower) and obscure biologically meaningful local alignments. Rationale: state annotation is the reduction step that turns the hard problem of comparing differently sized multi-track signal matrices into comparing two compressed state sequences.

## Related tools

- **ChromHMM** (Hidden Markov model that assigns discrete chromatin state labels to genomic regions based on signals of multiple epigenetic marks, producing the single-track state sequences used by EpiAlign)
- **EpiAlign** (Downstream consumer: performs local alignment (modified Smith–Waterman dynamic programming with state weights, εN=1.5, εD=1) on the annotated (and compressed) chromatin state sequences) — https://github.com/zzz3639/EpiAlign

## Evaluation signals

- Compression check: consecutive identical state labels must be collapsed to single labels; benchmark wall-clock time so uncompressed alignment is roughly 20x the compressed alignment time
- Sample consistency: the same state model/number of states (e.g. 7-state) must be used across samples being compared so state labels are comparable between sequences
- Region coverage: annotated sequences must span the queried regions — whole chromosomes for horizontal alignment (500 kb regions), or all 19,935 GENCODE v10 protein-coding gene regions (minus chromosome Y) for vertical alignment
- Downstream separability: real Roadmap epigenome regions should score above 0 on average in EpiAlign horizontal alignment while randomized negative-control scores stay close to 0
- Quiescent-state handling: verify that motifs/alignments involving the quiescent state 'o' are either handled per protocol (compressed away) or excluded where 'o' is not considered

## Limitations

- Likely noisy ChromHMM annotations can arise and need further biological validation (referenced as Supplementary section 12, not included in the provided content)
- Long stretches of a single state (notably quiescent/low states) are not useful for comparing chromatin state sequences (Supplementary Figure S1, not included); compression is required
- Alignment results depend on ChromHMM output; constructing specialized substitution matrices for chromatin states remains an open future research question
- No version numbers for ChromHMM are stated in the article or the EpiAlign repository, which has no changelog

## Evidence

- [intro] ChromHMM assigns discrete chromatin state labels to genomic regions based on signals of multiple epigenetic marks using a hidden Markov model (9).: "ChromHMM assigns discrete chromatin state labels to genomic regions based on signals of multiple epigenetic marks using a hidden Markov model (9)."
- [methods] We first apply a chromatin state annotation method (e.g. ChromHMM (9)) to encode multi-track epigenomic signals into single-track chromatin state sequences: "We first apply a chromatin state annotation method (e.g. ChromHMM (9)) to encode multi-track epigenomic signals into single-track chromatin state sequences"
- [methods] Second, we compress consecutive occurrences of the same state into a state label.: "Second, we compress consecutive occurrences of the same state into a state label."
- [intro] the NIH Roadmap project has released public epigenomic data of 127 human tissue and cell types (7). This database (release 9) contains a total of 2804 genome-wide epigenomic datasets: "the NIH Roadmap project has released public epigenomic data of 127 human tissue and cell types (7). This database (release 9) contains a total of 2804 genome-wide epigenomic datasets"
- [results] we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database: "we use all protein-coding genes in GENCODE v10 (28) that are compatible with the Roadmap database"
