---
name: hoogsteen-triplex-sequence-scanning
description: Use when the goal is to predict RNA:DNA:DNA triple-helix interactions
  in silico from an ssRNA transcript (FASTA) and a set of dsDNA regulatory regions
  (multi-FASTA), e.g. lncRNA-to-promoter binding screens.
when_to_use_negative:
- Input is protein-mediated RNA:DNA interaction data or you lack the actual nucleotide
  sequences — 3plex requires raw ssRNA/dsDNA FASTA sequences.
- You need a genome-wide categorical prediction of which lncRNAs or DNA sites are
  triplex-forming rather than per-site TPX coordinates — a deep-learning classifier
  like TriplexFPP addresses that task instead.
- The RNA sequence is expected to be heavily structured in the interacting region;
  masking by single-strandedness will exclude those nucleotides, so unmasked (ss0)
  mode or an alternative tool may be required.
edam_operation: http://edamontology.org/operation_0464
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3511
tools:
- name: 3plex
  role: Main scanner implementing Hoogsteen pairing rules (PATO algorithm), RNAplfold
    masking, and LongTarget stability scoring to return TPX predictions
  repo: https://github.com/molinerisLab/3plex
- name: RNAplfold
  role: Computes per-nucleotide single-strandedness from the ViennaRNA package; nucleotides
    below the cut-off are masked before scanning
- name: Triplexator
  role: Reference algorithm whose canonical Hoogsteen pairing rules and triplet definition
    are used for the triplex scanning step
- name: LongTarget
  role: Source of in vitro triplet thermal-stability data (He et al. denaturation
    experiments) used to score merged TTSs
- name: TriplexFPP
  role: Alternative CNN-based triplex prediction tool; used as a comparison, with
    poor agreement to 3plex predictions observed
  repo: https://github.com/yuuuuzhang/TriplexFPP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:049be06faa34c1f94c65b7afa7d0cb3d290c9a3317f508f2366df5059166e85a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  repo_url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - LongTarget
  - RNAplfold
  - TriplexFPP
  - Triplexator
  - ViennaRNA
  edam_operation: http://edamontology.org/operation_0464
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Hoogsteen triplex sequence scanning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scans ssRNA and dsDNA sequence pairs for triple-helix (TPX) formations using canonical Hoogsteen pairing rules, with RNAplfold-based single-strandedness masking and LongTarget-derived thermal stability scoring, as implemented in the 3plex tool.

## When to use

Use when the goal is to predict RNA:DNA:DNA triple-helix interactions in silico from an ssRNA transcript (FASTA) and a set of dsDNA regulatory regions (multi-FASTA), e.g. lncRNA-to-promoter binding screens.

## When NOT to use

- Input is protein-mediated RNA:DNA interaction data or you lack the actual nucleotide sequences — 3plex requires raw ssRNA/dsDNA FASTA sequences.
- You need a genome-wide categorical prediction of which lncRNAs or DNA sites are triplex-forming rather than per-site TPX coordinates — a deep-learning classifier like TriplexFPP addresses that task instead.
- The RNA sequence is expected to be heavily structured in the interacting region; masking by single-strandedness will exclude those nucleotides, so unmasked (ss0) mode or an alternative tool may be required.

## Inputs

- ssRNA sequence in FASTA format (single sequence, e.g. ssRNA.fa)
- dsDNA sequences in multi-FASTA format (e.g. promoter regions, dsDNA.fa)
- Optional parameters: min_length (>=5), error_rate, single_strandedness cutoff, consecutive_errors, guanine_rate, filter_repeat flag

## Outputs

- Tab-delimited gzipped file (*_tpx.stability.gz) of all TFO:TTS matches with columns: Sequence_ID, TFO_start, TFO_end, Duplex_ID, TTS_start, TTS_end, Score, Error_rate, Errors, Motif, Strand, Orientation, Guanine_rate, Stability, aln1-aln4

## How to apply

Run 3plex with an ssRNA FASTA (one sequence) and a dsDNA multi-FASTA: the PATO algorithm scans sequence pairs under Hoogsteen triplet rules (two Watson-Crick paired nucleotides in dsDNA plus one Hoogsteen-paired ssRNA nucleotide) and returns all TPXs satisfying constraints such as minimum length (default 10, allowed N>=5), error rate (default 20%), consecutive errors (default 1), and guanine rate (default 40%). Optionally mask RNA nucleotides below an RNAplfold single-strandedness cut-off (ss10/ss20/ss50, corresponding to masking 10%, 20% or 50% of the sequence; ss0 leaves the sequence unmasked, window of length 8) before scanning. Merge overlapping TPXs and score them with the normalised-stability scheme using in vitro triplet stability data from the LongTarget/He et al. denaturation experiments, divided by dsDNA length (undercounting scheme). Output is a tab-delimited *_tpx.stability.gz file listing TFO:TTS matches with coordinates, Score, Error_rate, Motif, Strand, Orientation, Guanine_rate, Stability and alignment columns.

## Related tools

- **3plex** (Main scanner implementing Hoogsteen pairing rules (PATO algorithm), RNAplfold masking, and LongTarget stability scoring to return TPX predictions) — https://github.com/molinerisLab/3plex
- **RNAplfold** (Computes per-nucleotide single-strandedness from the ViennaRNA package; nucleotides below the cut-off are masked before scanning)
- **Triplexator** (Reference algorithm whose canonical Hoogsteen pairing rules and triplet definition are used for the triplex scanning step)
- **LongTarget** (Source of in vitro triplet thermal-stability data (He et al. denaturation experiments) used to score merged TTSs)
- **TriplexFPP** (Alternative CNN-based triplex prediction tool; used as a comparison, with poor agreement to 3plex predictions observed) — https://github.com/yuuuuzhang/TriplexFPP

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Output file *_tpx.stability.gz exists and is tab-delimited with the 18 expected columns including Stability, Error_rate, Guanine_rate and alignment strings (aln1-aln4).
- All reported TPXs satisfy the chosen constraints: length >= min_length (default 10), error rate <= 20%, consecutive errors <= 1, guanine rate >= 40%.
- With a single-strandedness cut-off (e.g. ss50), no predicted TFO overlaps RNA nucleotides masked by RNAplfold; setting cutoff 0 (ss0) should yield a superset of predictions.
- Undercounting normalised-stability scoring should perform similarly to or slightly better than the overcounting alternative, reproducing the paper's expected behaviour (stability-derived measures beat simple match counts).
- End-to-end run completes on the provided test ssRNA.fa and dsDNA.fa (e.g. SRA1 against tss.1500_500 promoter subset) producing plausible TFO:TTS pairs like SRA1 149-159 vs CCDC27::chr1:3750949-3752950.

## Limitations

- Minimal TPX length of 6 gave no further improvement in a subsample of the parameter space, and allowing 1 vs up to 3 consecutive errors showed no significant AUC differences — these parameters are largely insensitive.
- TPXs tend to be shorter and more degenerated than previously expected, so overly strict length/guanine constraints may miss real interactions.
- TriplexFPP comparison showed no good agreement, and TriplexFPP failed to identify experimentally known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC) — cross-tool consensus should not be assumed.
- Docker runs require vsyscall enabled on host kernels >= 4.8, and input files must be accessible by the Docker user/group (139 exit status signals the vsyscall issue).

## Evidence

- [readme] ssRNA.fa          	The RNA sequence in FASTA format. The file must contain only one sequence.: [evidence span withheld — anchor status: external_doc]
