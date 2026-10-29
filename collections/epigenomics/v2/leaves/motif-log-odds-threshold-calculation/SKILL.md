---
name: motif-log-odds-threshold-calculation
description: Use when a positional motif-enrichment pipeline needs, per JASPAR PWM,
  a statistically calibrated log-odds cutoff (via MOODS) before scanning scored DNA
  sequences for motif matches.
when_to_use_negative:
- The motif library is not in JASPAR PWM matrix format or the background nucleotide
  composition of the sequences is unknown, since both are required to calibrate the
  log-odds distribution
- The goal is de novo motif discovery or motif assembly — MEPP currently functions
  with a fixed motif library only
- A presence/absence (ZOOPs-style) enrichment call per sequence is needed rather than
  position-resolved excess scores, e.g. when using methods like HOMER or MEIRLOP
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0749
tools:
- name: MEPP
  role: Implements MOODS-based motif match threshold setting (via --pcount, --pval,
    --bg) before scanning scored FASTA sequences to build positional heatmaps
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
  source_content_hash: sha256:1a55095107a9fa64e8053cb5d91e720346b924e47dd078ba9aeb3792efa25fab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# motif-log-odds-threshold-calculation

## Summary

Computes a JASPAR PWM-specific MOODS log-odds score threshold so that MEPP can record motif match excess scores max(0, score - threshold) when scanning one-hot encoded scored FASTA sequences, with the threshold governed by a pseudocount, p-value, and background nucleotide composition.

## When to use

Use when a positional motif-enrichment pipeline needs, per JASPAR PWM, a statistically calibrated log-odds cutoff (via MOODS) before scanning scored DNA sequences for motif matches.

## When NOT to use

- The motif library is not in JASPAR PWM matrix format or the background nucleotide composition of the sequences is unknown, since both are required to calibrate the log-odds distribution
- The goal is de novo motif discovery or motif assembly — MEPP currently functions with a fixed motif library only
- A presence/absence (ZOOPs-style) enrichment call per sequence is needed rather than position-resolved excess scores, e.g. when using methods like HOMER or MEIRLOP

## Inputs

- JASPAR-format PWM motif matrices file (e.g. from http://jaspar.genereg.net/downloads/ or the bundled HOMER-derived data/homer.motifs.txt)
- Pseudocount parameter (--pcount, default 0.0001)
- P-value parameter (--pval, default 0.0001)
- Background DNA composition as four floats (--bg, default 0.25 0.25 0.25 0.25)

## Outputs

- Per-motif MOODS log-odds threshold used to define significant motif matches during one-hot-encoded sequence scanning
- Motif-by-position heatmap cells holding excess scores max(0, score - threshold) (motif_heatmap.tsv)

## How to apply

For each PWM in the fixed JASPAR-format motif library, compute the MOODS log-odds score threshold that defines a significant motif match. Set the threshold from three user-tunable parameters: a pseudocount (default 0.0001) applied to the motif matrix, a p-value (default 0.0001) for the log-odds score distribution, and a 4-float background DNA composition (default 0.25 0.25 0.25 0.25). Rationale: this score-based principle avoids arbitrary threshold selection while controlling for sequence bias, which is central to MEPP's design. The resulting threshold is then used during scanning so each padded position records the excess score max(0, score - threshold), with a dual-orientation max rule taking the higher of forward and reverse-complement excess scores.

## Related tools

- **MEPP** (Implements MOODS-based motif match threshold setting (via --pcount, --pval, --bg) before scanning scored FASTA sequences to build positional heatmaps) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP from the GitHub repository) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored.fa --motifs jaspar_motifs.txt --out mepp_out --pcount 0.0001 --pval 0.0001 --bg 0.25 0.25 0.25 0.25
```

## Evaluation signals

- Each motif in the JASPAR library receives a finite log-odds threshold; no motif match should produce a negative excess score since cells store max(0, score - threshold)
- Varying --pval from 0.0001 should monotonically shift thresholds (lower p-value → higher threshold) and change heatmap cell intensities accordingly
- A custom --bg (four floats summing to 1, e.g. GC-rich composition) should produce thresholds distinct from the uniform 0.25 0.25 0.25 0.25 default for GC-skewed motifs
- The resulting motif_heatmap.tsv must have one row per motif and columns spanning the padded sequence length

## Limitations

- Threshold calibration depends on the assumed background composition; the default 0.25 0.25 0.25 0.25 may be inappropriate for GC-biased sequences, and --dgt filtering (default 100%) controls rejection of degenerate sequences
- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified for MEPP in the article

## Evidence

- [readme] Pseudocount for setting motif match threshold via MOODS. Default: 0.0001: [evidence span withheld — anchor status: external_doc]
- [readme] P-value for setting motif match threshold via MOODS: [evidence span withheld — anchor status: external_doc]
- [readme] Background DNA composition for MOODS threshold: [evidence span withheld — anchor status: external_doc]
- [intro] Score-based principle avoids arbitrary thresholds: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [other] For each JASPAR PWM, compute its MOODS log-odds score threshold used to define significant motif matches.: [evidence span withheld — anchor status: not_found]
