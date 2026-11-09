---
name: pwm-one-hot-sequence-encoding
description: Use when scored FASTA sequences centered on an anchor (e.g. TSS or motif)
  must be converted to one-hot tensors and scanned with JASPAR PWMs to produce a motif-by-position
  heatmap for downstream positional enrichment analysis.
when_to_use_negative:
- 'You need de novo motif discovery: MEPP currently functions only with a fixed motif
  library and cannot assemble new motifs (use mepp learn_motifs or a separate discovery
  tool instead).'
- Input sequences are not uniform-length or not centered on a common anchor point
  — positional profiles require aligned coordinates across all sequences.
- A binary presence/absence (ZOOPs) motif enrichment test is the goal, rather than
  a positional excess-score profile (MEPP deliberately scores excess over the MOODS
  threshold rather than zero-or-one occurrence).
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: MEPP
  role: Implements the one-hot encoding, MOODS-thresholded PWM scanning, dual-orientation
    max rule, and margin-based average pooling that produce the motif heatmap
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP from the GitHub repository
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:049061e062a3696e350a59b1f14a8370959e91ba020daf7a07c6f2e129ff9887
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# PWM one-hot sequence encoding and motif heatmap generation

## Summary

One-hot encodes uniform-length scored DNA sequences and scans them against a fixed JASPAR PWM library (MOODS log-odds thresholds) to build a motif-by-position excess-score heatmap, the input representation MEPP uses for positional enrichment profiling.

## When to use

Use when scored FASTA sequences centered on an anchor (e.g. TSS or motif) must be converted to one-hot tensors and scanned with JASPAR PWMs to produce a motif-by-position heatmap for downstream positional enrichment analysis.

## When NOT to use

- You need de novo motif discovery: MEPP currently functions only with a fixed motif library and cannot assemble new motifs (use mepp learn_motifs or a separate discovery tool instead).
- Input sequences are not uniform-length or not centered on a common anchor point — positional profiles require aligned coordinates across all sequences.
- A binary presence/absence (ZOOPs) motif enrichment test is the goal, rather than a positional excess-score profile (MEPP deliberately scores excess over the MOODS threshold rather than zero-or-one occurrence).

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (uniform-length sequences centered on an anchor point)
- Motif matrices file in JASPAR format (e.g. from the JASPAR website or the bundled data/homer.motifs.txt)
- Optional parameters: --margin (default 2), --pval (default 0.0001), --pcount (default 0.0001), --bg (default 0.25 0.25 0.25 0.25), --dgt (default 100), --orientations (default '+,+/-' or '+/-')

## Outputs

- Motif-by-position excess-score heatmap matrix (one row per motif, columns spanning the padded sequence length), written to motif_heatmap.tsv
- Tensorflow dataset of one-hot encoded sequences (retained only with --keepdata)

## How to apply

Load the uniform-length scored DNA sequences (headers of the form '>sequence_name sequence_score') and the fixed JASPAR-format PWM library, one-hot encoding each sequence for motif scanning; reject sequences whose degenerate (non-A/C/G/T) fraction exceeds the --dgt threshold (default 100%). For each PWM, compute a MOODS log-odds score threshold (default p-value 0.0001, pseudocount 0.0001, uniform background 0.25/0.25/0.25/0.25) and scan each one-hot sequence, recording at each padded position the excess score max(0, score - threshold). Apply the dual-orientation max rule so each heatmap cell holds the higher excess score between forward and reverse-complement scans (controlled by --orientations, e.g. '+/-'). Optionally apply average pooling along the position axis with window w = 1 + 2m, where m is the user-defined --margin (default 2, i.e. 'blur' 2 bases on either side of each motif match) to smooth the heatmap for inexact motif positioning. Write the motif-by-position matrix (one row per motif, columns spanning the padded sequence length) and verify the table schema before downstream MEPP profiling.

## Related tools

- **MEPP** (Implements the one-hot encoding, MOODS-thresholded PWM scanning, dual-orientation max rule, and margin-based average pooling that produce the motif heatmap) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP from the GitHub repository) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored_sequences.fa --motifs jaspar_motifs.txt --out mepp_out --margin 2 --orientations "+/-" --pval 0.0001
```

## Evaluation signals

- motif_heatmap.tsv has exactly one row per motif in the JASPAR library and columns spanning the padded sequence length
- Every heatmap cell value is >= 0, since values are excess scores max(0, score - threshold) with the MOODS threshold at p-value 0.0001
- Sequences with degenerate (non-A/C/G/T) fraction above the --dgt percentage are excluded from the dataset
- With default --margin 2, the smoothing window along the position axis equals w = 1 + 2m = 5 positions
- Cells under the dual-orientation ('+/-') rule equal the maximum of the forward and reverse-complement excess scores at that position

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- Large --perms values (default 1000) for permutation testing can lead to significant GPU memory usage; use --nogpu on CPU-only systems
- Motif matches are 'blurred' by the margin (default 2 bases each side), so heatmap resolution is intentionally smoothed and single-base positional resolution is not preserved

## Evidence

- [other] 1. Load the uniform-length scored DNA sequences and the fixed JASPAR PWM motif library, one-hot encoding each sequence for motif scanning.: [evidence span withheld — anchor status: not_found]
- [other] The average pooling window applied to the motif score heatmap rows is computed as w = 1 + 2m, where m is a user-defined motif-margin: [evidence span withheld — anchor status: not_found]
- [readme] --margin INTEGER   Number of bases along either side of motif to "blur" motif matches for smoothing.: "--margin INTEGER                Number of bases along either side of motif
                                      to "blur" motif matches for smoothing.
                                      Default: 2"
- [readme] --pval FLOAT   P-value for setting motif match threshold via MOODS. Default: 0.0001: "--pval FLOAT                    P-value for setting motif match threshold
                                      via MOODS.  Default: 0.0001"
- [readme] --dgt INTEGER   Percentage of sequence that can be degenerate (Not A, C, G, or T) before being rejected from the analysis.: "--dgt INTEGER                   Percentage of sequence that can be
                                      degenerate (Not A, C, G, or T) before being
                                      rejected"
