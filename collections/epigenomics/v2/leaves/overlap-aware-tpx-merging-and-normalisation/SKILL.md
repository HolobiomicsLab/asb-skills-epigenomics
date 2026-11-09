---
name: overlap-aware-tpx-merging-and-normalisation
description: 'Use when raw PATO/3plex TFO:TTS predictions must be consolidated and
  ranked: overlapping TPXs need merging, and a duplex-length-normalised thermal-stability
  score is needed for cross-duplex comparison.'
when_to_use_negative:
- When the goal is RNA secondary-structure prediction itself (RNAplfold -u unpaired
  probabilities) rather than scoring already-predicted TPXs
- When raw RNA:DNA interaction peaks still need replicate filtering (IDR conservative/optimal,
  Overlap, Top1000, SPP q-value < 0.05) before they can serve as evaluation ground
  truth
- When a machine-learning triplex classifier is required instead of rule- and stability-based
  scoring (e.g. TriplexFPP CNN models)
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0099
tools:
- name: 3plex
  role: Implements the TPX merging, LongTarget-derived thermal-stability scoring and
    dsDNA-length normalisation, emitting *_tpx.stability.gz
  repo: https://github.com/molinerisLab/3plex
- name: LongTarget
  role: Source of the in vitro triplet denaturation data (He et al. 2015) used to
    score stability of merged TTSs
- name: RNAplfold
  role: Upstream masking of paired RNA nucleotides (ViennaRNA -u option, window of
    8) that constrains which TPXs enter the merging/scoring stage
- name: ViennaRNA
  role: Package providing RNAplfold used for RNA secondary structure prediction before
    scanning
- name: PATO
  role: Scanner returning candidate triplexes under user-defined constraints that
    feed the merge-and-score step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:4eef02d633ccbd5568e9cb6d6fd735b6e19f7ae0b6a3c4859324658ecfac2c2c
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
  - PATO
  - RNAplfold
  - Triplexator
  - ViennaRNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# overlap-aware-tpx-merging-and-normalisation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scoring stage of the 3plex pipeline: overlapping triplex (TPX) predictions are merged, assigned a thermal stability score from the LongTarget/He et al. in vitro triplet denaturation data, and normalised by dsDNA length (undercounting scheme) so that RNA:DNA:DNA interactions are comparable across duplexes.

## When to use

Use when raw PATO/3plex TFO:TTS predictions must be consolidated and ranked: overlapping TPXs need merging, and a duplex-length-normalised thermal-stability score is needed for cross-duplex comparison.

## When NOT to use

- When the goal is RNA secondary-structure prediction itself (RNAplfold -u unpaired probabilities) rather than scoring already-predicted TPXs
- When raw RNA:DNA interaction peaks still need replicate filtering (IDR conservative/optimal, Overlap, Top1000, SPP q-value < 0.05) before they can serve as evaluation ground truth
- When a machine-learning triplex classifier is required instead of rule- and stability-based scoring (e.g. TriplexFPP CNN models)

## Inputs

- TPX scan results from PATO/3plex (TFO:TTS matches with coordinates, error rate, motif, guanine rate)
- RNAplfold single-strandedness-masked ssRNA FASTA (cut-off ss0/ss10/ss20/ss50)
- dsDNA multi-FASTA (e.g. promoter sequences) providing duplex lengths for normalisation
- Thermal stability lookup from the LongTarget collection of triplex denaturation experiments (He et al. 2015, Supplementary Table 5)

## Outputs

- Tab-delimited *_tpx.stability.gz listing all TFO:TTS matches with columns: Sequence_ID, TFO_start, TFO_end, Duplex_ID, TTS_start, TTS_end, Score, Error_rate, Errors, Motif, Strand, Orientation, Guanine_rate, Stability, and 4-line alignment blocks
- Duplex-length-normalised stability Score per TPX suitable for ranking and AUC evaluation against RADICL-seq peak sets

## How to apply

After PATO scanning (with or without RNAplfold single-strandedness masking at ss10/ss20/ss50 cut-offs), collect all TPXs passing user constraints (min length default 10, error rate default 20%, consecutive errors default 1, guanine rate default 40%) and merge those whose TTS coordinates overlap on the same duplex, retaining the maximal thermal stability among merged TTSs using the He et al. in vitro triplet stability table. Compute the final score by dividing this stability value by the dsDNA length — the 'undercounting' scheme, which the authors found performs similarly to or slightly better than the 'overcounting' alternative and outperforms simple match counts. Emit per-TFO:TTS rows in the tab-delimited *_tpx.stability.gz output containing Score, Error_rate, Motif, Guanine_rate and Stability columns. Judge correctness by checking that merged entries share TTS coordinates, that Stability values are plausible denaturation-derived temperatures (e.g. ~30-40 in the README example), and that the AUC behaviour on RADICL-seq-derived positives matches the paper's expectation for stability-derived scoring.

## Related tools

- **3plex** (Implements the TPX merging, LongTarget-derived thermal-stability scoring and dsDNA-length normalisation, emitting *_tpx.stability.gz) — https://github.com/molinerisLab/3plex
- **LongTarget** (Source of the in vitro triplet denaturation data (He et al. 2015) used to score stability of merged TTSs)
- **RNAplfold** (Upstream masking of paired RNA nucleotides (ViennaRNA -u option, window of 8) that constrains which TPXs enter the merging/scoring stage)
- **ViennaRNA** (Package providing RNAplfold used for RNA secondary structure prediction before scanning)
- **PATO** (Scanner returning candidate triplexes under user-defined constraints that feed the merge-and-score step)

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Output file is *_tpx.stability.gz and its header contains Score, Stability, Error_rate, Motif, Guanine_rate columns
- Merged entries share overlapping TTS_start/TTS_end coordinates on the same Duplex_ID and carry the maximal stability among the merged TTSs
- Score equals stability divided by dsDNA length, so scores are comparable across duplexes of different lengths
- Example values from the README show Stability ~32.8-36.4 for 11-bp TPXs with Error_rate 0.2 and Guanine_rate 0.4-0.5, Motif 'Y'
- Undercounting normalised scoring reproduces the paper's expected behaviour: similar to or slightly better than the overcounting scheme, and better than match-count scores when evaluated as AUC on RADICL-seq positive peaks

## Limitations

- Stability scores derive from in vitro denaturation experiments (He et al. 2015) and may not fully capture in vivo triplex behaviour
- The undercounting scheme divides by dsDNA length and thus undercounts long duplexes relative to the overcounting alternative
- Minimal TPX length of 6 gave no further improvement, and allowing 1 vs up to 3 consecutive errors made no significant difference in average AUC, so parameter tuning beyond defaults yields little gain
- 3plex predictions showed no good agreement with TriplexFPP, which also missed experimentally known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC)

## Evidence

- [readme] output format with Score and Stability columns: [evidence span withheld — anchor status: external_doc]
