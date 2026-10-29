---
name: average-pooled-heatmap-smoothing
description: Use when building or tuning a MEPP motif score heatmap from scored FASTA
  sequences and JASPAR PWMs and you need to set the positional blur/smoothing window
  via the --margin parameter.
when_to_use_negative:
- You need raw, unsmoothed per-position excess scores rather than positionally blurred
  motif densities — pooling discards single-base resolution
- De novo motif recognition or assembly is the goal; MEPP currently functions with
  a fixed motif library
- Input sequences are not uniform-length and centered on an anchor point (e.g. TSS
  or motif), since positional profiles are computed relative to such centers
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0749
tools:
- name: MEPP
  role: Implements the one-hot encoding, MOODS-thresholded PWM scanning, dual-orientation
    max rule, and average pooling (window 1 + 2m) that produce the smoothed motif
    heatmap
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP used to obtain the heatmap-generation module
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:d80cd297cd9c588e05af87df17e26034554ec16a6d0329f2529abc8feb843d04
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
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# average-pooled-heatmap-smoothing

## Summary

MEPP's heatmap generation module smooths the motif-by-position score heatmap rows with average pooling of window w = 1 + 2m (m = user-defined --margin, default 2), blurring motif matches along the position axis to account for inexact motif positioning.

## When to use

Use when building or tuning a MEPP motif score heatmap from scored FASTA sequences and JASPAR PWMs and you need to set the positional blur/smoothing window via the --margin parameter.

## When NOT to use

- You need raw, unsmoothed per-position excess scores rather than positionally blurred motif densities — pooling discards single-base resolution
- De novo motif recognition or assembly is the goal; MEPP currently functions with a fixed motif library
- Input sequences are not uniform-length and centered on an anchor point (e.g. TSS or motif), since positional profiles are computed relative to such centers

## Inputs

- Scored FASTA file with sequence headers of the form '>sequence_name sequence_score'
- Motif matrices file in JASPAR format (e.g. from http://jaspar.genereg.net/downloads/ or data/homer.motifs.txt)
- --margin INTEGER: number of bases on either side of a motif match to blur (default 2)
- MOODS threshold settings: --pval, --pcount, --bg (4 background floats)
- --orientations setting, e.g. '+/-' for forward or reverse-complement matching

## Outputs

- Smoothed motif-by-position heatmap matrix written to motif_heatmap.tsv (one row per motif, columns spanning the padded sequence length)
- Cell values are average-pooled excess scores, max(0, score - MOODS threshold), maxed over orientations before pooling

## How to apply

First, one-hot encode each uniform-length scored DNA sequence from the scored FASTA (headers of the form '>sequence_name sequence_score') and load the fixed JASPAR PWM library. For each PWM, compute a MOODS log-odds threshold (defaults: --pval 0.0001, --pcount 0.0001, background 0.25 0.25 0.25 0.25) and scan every sequence, recording at each padded position the excess score max(0, score - threshold). Fill each heatmap cell with the higher excess score between forward and reverse-complement scans ('+/-' orientations). Then apply average pooling along the position axis with window w = 1 + 2m, where m is the --margin option ('number of bases along either side of motif to blur motif matches for smoothing', default 2, giving a 5 bp window); this symmetric odd-sized window preserves the central position while averaging m bases on either side. Write the smoothed motif-by-position matrix to motif_heatmap.tsv and verify it has one row per motif and columns spanning the padded sequence length.

## Related tools

- **MEPP** (Implements the one-hot encoding, MOODS-thresholded PWM scanning, dual-orientation max rule, and average pooling (window 1 + 2m) that produce the smoothed motif heatmap) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP used to obtain the heatmap-generation module) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa scored.fa --motifs jaspar_motifs.txt --out mepp_out --margin 2 --orientations +/- --pval 0.0001
```

## Evaluation signals

- motif_heatmap.tsv has exactly one row per motif in the input JASPAR library and columns spanning the padded sequence length
- The pooling window is symmetric and odd-sized: w = 1 + 2m; with the default --margin 2 the window is 5 positions
- Every smoothed cell value is bounded above by the pre-pooling maximum excess score and below by 0 (clipped excess max(0, score - threshold))
- Cells reflect the dual-orientation max rule: each cell holds the higher of forward and reverse-complement excess scores before pooling
- Changing --margin visibly changes the positional blur: margin m=0 yields w=1 (no pooling), larger m yields proportionally wider smoothing

## Limitations

- MEPP currently functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage, so smoothing runs may fail if GPU jobs crowd each other out of memory (use --nogpu or tune --gjobs)
- No software version specified for MEPP; no changelog found for the repository

## Evidence

- [readme] --margin INTEGER                Number of bases along either side of motif
                                      to "blur" motif matches for smoothing.
                                      Default: 2: [evidence span withheld — anchor status: external_doc]
- [other] The average pooling window applied to the motif score heatmap rows is computed as w = 1 + 2m, where m is a user-defined motif-margin: [evidence span withheld — anchor status: not_found]
- [supplementary] The code for MEPP is available from its Github repository at https://github.com/npdeloss/mepp, and can be installed through pip: "The code for MEPP is available from its Github repository at https://github.com/npdeloss/mepp, and can be installed through pip"
- [discussion] MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs based: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work"
