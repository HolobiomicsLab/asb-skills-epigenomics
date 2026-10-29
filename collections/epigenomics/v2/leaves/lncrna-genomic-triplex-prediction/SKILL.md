---
name: lncrna-genomic-triplex-prediction
description: Use when predicting RNA:DNA:DNA triplex interactions between an lncRNA
  transcript (ssRNA FASTA) and genomic/promoter regions (dsDNA multi-FASTA), needing
  Hoogsteen-rule scanning, RNA secondary-structure masking, and thermal-stability
  scoring.
when_to_use_negative:
- 'You need a classification of triplex-forming potential from a CNN trained on experimental
  TPX data rather than rule-based scanning — use TriplexFPP instead (note: 3plex and
  TriplexFPP predictions did not agree well, and TriplexFPP failed to identify known
  TPX-forming lncRNAs such as CDKN2B-AS1, MEG3, NEAT1 and TERC).'
- Input ssRNA.fa contains more than one sequence — the 3plex required-argument format
  demands a single RNA sequence.
- The question is peak reproducibility or replicate handling of RNA:DNA interaction
  datasets (IDR conservative/optimal, Overlap, Top1000 filtering) rather than triplex
  prediction itself.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: 3plex
  role: 'Main pipeline: PATO triplex scanning, RNAplfold masking and LongTarget-based
    stability scoring; run via Docker image imolineris/3plex:v1.0'
  repo: https://github.com/molinerisLab/3plex
- name: RNAplfold
  role: ViennaRNA tool computing per-nucleotide single-strandedness used to exclude
    ssRNA nucleotides paired in secondary structure before scanning
- name: Triplexator
  role: Source of the canonical Hoogsteen pairing rules (two Watson-Crick nucleotides
    in dsDNA plus one Hoogsteen nucleotide in ssRNA) implemented by the PATO scanner
- name: LongTarget
  role: Collection of triplex denaturation experiments (He et al., 2015) supplying
    the in vitro triplet thermal-stability data used for normalised-stability scoring
- name: TriplexFPP
  role: Alternative deep-learning predictor (CNN-based triplex lncRNA and triplex
    DNA site models) used as a comparison tool in the benchmark
  repo: https://github.com/yuuuuzhang/TriplexFPP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:12a77580aa06ef0f4a5a5a4aa6a7d3161e184e5e5b1a6e1ecf2cdd15be0bf89c
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# lncrna-genomic-triplex-prediction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Predict ssRNA:dsDNA triple-helix (TPX) interactions in silico with 3plex, a pipeline integrating Triplexator-style Hoogsteen pairing rules, RNAplfold single-strandedness masking, and LongTarget-derived thermal stability scoring for triplex-forming lncRNAs.

## When to use

Use when predicting RNA:DNA:DNA triplex interactions between an lncRNA transcript (ssRNA FASTA) and genomic/promoter regions (dsDNA multi-FASTA), needing Hoogsteen-rule scanning, RNA secondary-structure masking, and thermal-stability scoring.

## When NOT to use

- You need a classification of triplex-forming potential from a CNN trained on experimental TPX data rather than rule-based scanning — use TriplexFPP instead (note: 3plex and TriplexFPP predictions did not agree well, and TriplexFPP failed to identify known TPX-forming lncRNAs such as CDKN2B-AS1, MEG3, NEAT1 and TERC).
- Input ssRNA.fa contains more than one sequence — the 3plex required-argument format demands a single RNA sequence.
- The question is peak reproducibility or replicate handling of RNA:DNA interaction datasets (IDR conservative/optimal, Overlap, Top1000 filtering) rather than triplex prediction itself.

## Inputs

- ssRNA.fa: RNA sequence in FASTA format (must contain only one sequence)
- dsDNA.fa: DNA sequences in multi-FASTA format (e.g. promoter/TTS regions)
- Optional: single-strandedness cut-off choice (ss0/ss10/ss20/ss50), filter_repeat flag, min/max length, error rate, consecutive errors, guanine rate

## Outputs

- One tab-delimited *_tpx.stability.gz file listing all TFO:TTS matches with columns: Sequence_ID, TFO_start, TFO_end, Duplex_ID, TTS_start, TTS_end, Score, Error_rate, Errors, Motif, Strand, Orientation, Guanine_rate, Stability, aln1-aln4
- Optional --RNA2D_out output: RNAplfold modified z-scores per path

## How to apply

Run the 3plex pipeline (Docker image imolineris/3plex:v1.0) on a single-sequence ssRNA.fa and a multi-FASTA dsDNA.fa, using the PATO algorithm to scan for triplexes satisfying user constraints (minimum length, default 10; error rate, default 20%; consecutive errors, default 1; guanine rate in the TTS, default 40%). Before scanning, optionally mask RNA nucleotides below an RNAplfold single-strandedness cut-off (ss10, ss20, ss50, masking 10%, 20% or 50% of the sequence; ss0 leaves it unmasked, computed with unpaired window of size 8 centred on each position). Score merged TPXs with the normalised-stability scheme: merge overlapping TPXs, keep the maximal thermal stability among merged TTSs using in vitro triplet stability data from the LongTarget/He et al. denaturation experiments (Supplementary Table 5), divided by the dsDNA length (undercounting scheme). The output is a tab-delimited *_tpx.stability.gz file listing all TFO:TTS matches with coordinates, score, error rate, motif, strand, orientation, guanine rate, stability and alignments.

## Related tools

- **3plex** (Main pipeline: PATO triplex scanning, RNAplfold masking and LongTarget-based stability scoring; run via Docker image imolineris/3plex:v1.0) — https://github.com/molinerisLab/3plex
- **RNAplfold** (ViennaRNA tool computing per-nucleotide single-strandedness used to exclude ssRNA nucleotides paired in secondary structure before scanning)
- **Triplexator** (Source of the canonical Hoogsteen pairing rules (two Watson-Crick nucleotides in dsDNA plus one Hoogsteen nucleotide in ssRNA) implemented by the PATO scanner)
- **LongTarget** (Collection of triplex denaturation experiments (He et al., 2015) supplying the in vitro triplet thermal-stability data used for normalised-stability scoring)
- **TriplexFPP** (Alternative deep-learning predictor (CNN-based triplex lncRNA and triplex DNA site models) used as a comparison tool in the benchmark) — https://github.com/yuuuuzhang/TriplexFPP

## Examples

```
docker run -u `id -u`:`id -g` -it --rm -v $PWD:$PWD imolineris/3plex:v1.0 $PWD/ssRNA.fa $PWD/dsDNA.fa $PWD/results_3plex
```

## Evaluation signals

- Output file *_tpx.stability.gz exists and is tab-delimited with the expected 18 columns (Sequence_ID through aln4), each row containing TFO:TTS coordinates, motif, strand, orientation, guanine_rate and stability.
- Default parameters are respected unless overridden: min_length 10 (values >=5 allowed), error_rate 20, single_strandedness_cutoff 0, consecutive_errors 1, guanine_rate 40.
- Undercounting normalised-stability scoring reproduces the paper's expected behaviour: performing similarly to or slightly better than the overcounting alternative.
- End-to-end validation on the example inputs (test/SRA1.fa and the MANE GRCh38 promoter subset dsDNA FASTA) yields TPX rows such as SRA1:149-159 against CCDC27::chr1:3750949-3752950 with error rate 0.2 and guanine rate 0.4.
- Selected ss cut-off masks the intended fraction of the sequence (10%, 20% or 50% for ss10/ss20/ss50); ss0 leaves the sequence unmasked.

## Limitations

- No significant differences in average AUC were observed allowing just 1 or up to 3 consecutive errors, and a minimal TPX length of 6 gave no further improvement in a subsample of the parameter space, so fine parameter tuning in these ranges is unlikely to help.
- 3plex and TriplexFPP predictions did not agree well; TriplexFPP missed experimentally known TPX-forming lncRNAs (CDKN2B-AS1, lncSmad7, MEG3, NEAT1, TERC), and TriplexAligner's AUC differed considerably from author-reported values under this benchmark strategy.
- Operational caveats from the README: ssRNA.fa must contain exactly one sequence; on hosts with linux kernel >= 4.8 the Docker container may exit with status 139 unless vsyscall is enabled; input files must be accessible to the Docker user/group.
- TPXs were found to tend to be shorter and more degenerated than previously expected, so overly strict length/error constraints may miss real interactions.

## Evidence

- [readme] ssRNA.fa          	The RNA sequence in FASTA format. The file must contain only one sequence.: [evidence span withheld — anchor status: external_doc]
