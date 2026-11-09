---
name: positional-motif-heatmap-construction
description: Use when you have scored DNA sequences (FASTA headers of the form '>name
  score') centered on anchor points and a fixed JASPAR PWM library, and need a motif-by-position
  heatmap of excess match scores relative to the anchors.
when_to_use_negative:
- 'You need de novo motif discovery: ''MEPP currently functions with a fixed motif
  library'' and de novo recognition/assembly is left to future work'
- Input sequences are not scored/centered on an anchor (e.g. TSS or motif) — MEPP
  profiles enrichment relative to anchor points and headers must carry a numeric score
- You need a binary presence/absence ZOOPs-style enrichment call (as in MEIRLOP/HOMER)
  rather than a continuous positional excess-score profile
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0092
tools:
- name: MEPP
  role: 'Main tool: one-hot encodes scored sequences, scans with JASPAR PWMs, builds
    and smooths the motif score heatmap'
  repo: https://github.com/npdeloss/mepp
- name: MOODS
  role: Computes the per-PWM log-odds match threshold (from --pval/--pcount/--bg)
    used to define significant motif matches and the excess score
- name: pip
  role: Installation channel for the mepp package
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:b060fd02dad73ce5cceee03c3853e972b66f5300d4d4ed1f84d7f3ff55a2bd1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - MOODS
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# positional-motif-heatmap-construction

## Summary

Constructs the MEPP motif-by-position score heatmap by one-hot encoding scored FASTA sequences, scanning them with JASPAR PWMs at MOODS-derived log-odds thresholds, and recording the max excess score over forward/reverse orientations, optionally smoothed with an average-pooling window of w = 1 + 2m (m = --margin, default 2).

## When to use

Use when you have scored DNA sequences (FASTA headers of the form '>name score') centered on anchor points and a fixed JASPAR PWM library, and need a motif-by-position heatmap of excess match scores relative to the anchors.

## When NOT to use

- You need de novo motif discovery: 'MEPP currently functions with a fixed motif library' and de novo recognition/assembly is left to future work
- Input sequences are not scored/centered on an anchor (e.g. TSS or motif) — MEPP profiles enrichment relative to anchor points and headers must carry a numeric score
- You need a binary presence/absence ZOOPs-style enrichment call (as in MEIRLOP/HOMER) rather than a continuous positional excess-score profile

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score'
- JASPAR-format PWM motif matrices file (e.g. from JASPAR downloads or data/homer.motifs.txt in the repo)
- Optional: background DNA composition as four floats (default 0.25 0.25 0.25 0.25)

## Outputs

- motif_heatmap.tsv — motif-by-position matrix of excess match scores (one row per motif, columns spanning the padded sequence length)
- MEPP positional enrichment plots (png/svg, default 10x10 inches, 300 dpi)

## How to apply

Install MEPP via pip ('pip install mepp' or 'pip install git+https://github.com/npdeloss/mepp@main') and load the scored FASTA plus the JASPAR-format motif matrices file. One-hot encode each uniform-length sequence and set the motif match threshold per PWM with MOODS using --pval 0.0001, --pcount 0.0001, and background composition (--bg, default 0.25 for each base). Scan every sequence with every PWM, recording at each padded position the excess score max(0, score - threshold), and apply the dual-orientation max rule so each heatmap cell holds the higher excess score between the '+' forward and reverse-complement ('+/-' orientations flag) scans. Smooth heatmap rows with average pooling using window w = 1 + 2m, where m is the --margin parameter ('blur' of motif matches along either side of the motif, default 2), adjusting positional resolution. Scale match brightness adaptively via --sigma (default 0.5, max brightness at sigma * std of nonzero match scores) and write the motif-by-position matrix (one row per motif, columns spanning the padded sequence length) to motif_heatmap.tsv.

## Related tools

- **MEPP** (Main tool: one-hot encodes scored sequences, scans with JASPAR PWMs, builds and smooths the motif score heatmap) — https://github.com/npdeloss/mepp
- **MOODS** (Computes the per-PWM log-odds match threshold (from --pval/--pcount/--bg) used to define significant motif matches and the excess score)
- **pip** (Installation channel for the mepp package) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored.fa --motifs jaspar_motifs.txt --out mepp_out --margin 2 --orientations +/- --pval 0.0001 --pcount 0.0001
```

## Evaluation signals

- motif_heatmap.tsv has exactly one row per motif in the PWM library and columns spanning the padded sequence length
- Every heatmap cell value is >= 0 because it stores max(0, score - MOODS threshold) under the dual-orientation max rule
- Smoothing window equals 1 + 2m for the chosen --margin (default m=2 gives window 5), verifiable by re-running with a different --margin and checking row resolution
- Thresholds are reproducible from MOODS with the recorded --pval 0.0001, --pcount 0.0001, and --bg parameters
- Heatmap brightness saturates at sigma * std of the nonzero match scores (default sigma 0.5); changing --sigma changes pixel intensity but not the underlying tsv values

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version specified for MEPP and no changelog found for the repository
- Permutation testing (--perms, default 1000) and batched Tensorflow datasets (--batch, default 1000) can consume significant GPU memory; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail
- Sequences exceeding --dgt percent degenerate bases (non-A/C/G/T) are rejected from the analysis

## Evidence

- [methods] To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [readme] --margin INTEGER                Number of bases along either side of motif
                                      to "blur" motif matches for smoothing.
                                      Default: 2: [evidence span withheld — anchor status: external_doc]
- [readme] --orientations TEXT             Comma-separated list of motif orientations
                                      to analyze for CPU multiprocessing.: [evidence span withheld — anchor status: external_doc]
- [readme] --pval FLOAT                    P-value for setting motif match threshold
                                      via MOODS. Default: 0.0001: [evidence span withheld — anchor status: external_doc]
- [discussion] MEPP currently functions with a fixed motif library.: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs"
