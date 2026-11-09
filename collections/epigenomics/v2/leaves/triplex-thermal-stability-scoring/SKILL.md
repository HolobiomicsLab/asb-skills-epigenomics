---
name: triplex-thermal-stability-scoring
description: Use when predicting ssRNA:dsDNA triple-helix interactions and you need
  to rank TFO:TTS matches by in vitro thermal stability instead of raw match counts,
  e.g. within a 3plex/Triplexator-style pipeline.
when_to_use_negative:
- You only need triplex presence/absence or lncRNA classification by deep learning
  — TriplexFPP (CNN on experimental TPX features) is a different approach, and the
  paper found no good agreement between 3plex and TriplexFPP predictions
- You require protein-mediated RNA:DNA interactions — this skill addresses only non-protein-mediated
  (NPM) triplex formation such as measured by RADICL-seq
edam_operation: http://edamontology.org/operation_0475
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0654
tools:
- name: 3plex
  role: Implements the thermal stability scoring of predicted TPXs based on experimentally
    in vitro determined triplet stability data; docker image imolineris/3plex:v1.0
  repo: https://github.com/molinerisLab/3plex
- name: LongTarget
  role: Source of the triplex denaturation experiment collection used to derive triplet
    thermal stability values (He et al., 2015)
- name: RNAplfold
  role: Computes per-nucleotide ssRNA single-strandedness used to mask paired nucleotides
    before stability-scored scanning
- name: ViennaRNA
  role: Package providing RNAplfold for RNA secondary structure prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:ebfa1e67356c6c8d15e77d02b1b85de2c11030d43edccc898c411af5dd5d746f
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
  - Triplexator
  - ViennaRNA
  edam_operation: http://edamontology.org/operation_0475
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# triplex-thermal-stability-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores predicted RNA:DNA triplexes (TPXs) using experimentally determined triplet thermal stability data from the LongTarget collection of triplex denaturation experiments (He et al., 2015), replacing simple match counts with a stability-derived score. The 3plex paper reports better prediction performance using stability-derived measures than match-count scoring.

## When to use

Use when predicting ssRNA:dsDNA triple-helix interactions and you need to rank TFO:TTS matches by in vitro thermal stability instead of raw match counts, e.g. within a 3plex/Triplexator-style pipeline.

## When NOT to use

- You only need triplex presence/absence or lncRNA classification by deep learning — TriplexFPP (CNN on experimental TPX features) is a different approach, and the paper found no good agreement between 3plex and TriplexFPP predictions
- You require protein-mediated RNA:DNA interactions — this skill addresses only non-protein-mediated (NPM) triplex formation such as measured by RADICL-seq

## Inputs

- ssRNA transcript sequence in FASTA format (single sequence, e.g. SRA1.fa)
- dsDNA genomic/promoter sequences in multi-FASTA format
- In vitro triplet stability data from the LongTarget triplex denaturation experiment collection (He et al., Supplementary Table 5)
- Optional: RNAplfold single-strandedness probabilities for masking cut-offs ss10/ss20/ss50

## Outputs

- Tab-delimited *_tpx.stability.gz file listing TFO:TTS matches with columns: Sequence_ID, TFO_start, TFO_end, Duplex_ID, TTS_start, TTS_end, Score, Error_rate, Errors, Motif, Strand, Orientation, Guanine_rate, Stability, aln1-aln4
- Normalised-stability scores per TFO (stability divided by dsDNA length, undercounting scheme)

## How to apply

After scanning sequence pairs for TPXs with the PATO algorithm under Hoogsteen pairing rules (constraints: min length, error rate, consecutive errors, guanine rate), merge overlapping TPXs and keep the maximal thermal stability among merged TTSs, using the in vitro triplet stability data from He et al. derived via the LongTarget collection of triplex denaturation experiments. Apply the normalised-stability 'undercounting' scheme by dividing by the dsDNA length; this was shown to perform similarly to or slightly better than the 'overcounting' alternative. Optionally mask RNA nucleotides below a single-strandedness cut-off (ss0/ss10/ss20/ss50) computed with RNAplfold before scanning. The stability value is reported per match in the *_tpx.stability.gz output (e.g. 'Stability' column values like 32.8, 36.4) alongside Score, Error_rate, Motif, and Guanine_rate.

## Related tools

- **3plex** (Implements the thermal stability scoring of predicted TPXs based on experimentally in vitro determined triplet stability data; docker image imolineris/3plex:v1.0) — https://github.com/molinerisLab/3plex
- **LongTarget** (Source of the triplex denaturation experiment collection used to derive triplet thermal stability values (He et al., 2015))
- **RNAplfold** (Computes per-nucleotide ssRNA single-strandedness used to mask paired nucleotides before stability-scored scanning)
- **ViennaRNA** (Package providing RNAplfold for RNA secondary structure prediction)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Output *_tpx.stability.gz contains a Stability column with plausible denaturation-derived values (e.g. 32.8, 36.4 in the README example rows)
- Undercounting normalised-stability scoring reproduces the paper's expected behaviour: performing similarly to or slightly better than the overcounting alternative
- Stability-derived scoring outperforms (or at least matches) simple match-count scores on RADICL-seq Top1000 positive peaks vs randomised negative regions (AUC comparison)
- Every reported TFO:TTS pair satisfies user constraints: min length ≥5 (default 10), error rate ≤20% (default), ≤1 consecutive error (default), guanine rate ≥40% (default)
- Pipeline runs end-to-end on lncRNA and DNA input sequences without errors

## Limitations

- Thermal stability is an in vitro-derived estimate (He et al. 2015 denaturation experiments) and may not fully reflect in vivo triplex formation
- Comparison with TriplexFPP showed no good agreement, and TriplexFPP failed to identify known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC) — cross-tool consensus is not guaranteed
- Minimal TPX length of 6 gave no further improvement, and allowing 1 vs up to 3 consecutive errors showed no significant difference in average AUC — parameter tuning has diminishing returns
- Docker runs require vsyscall enabled on hosts with linux-kernel >= 4.8, and input files must be accessible by the Docker user/group

## Evidence
