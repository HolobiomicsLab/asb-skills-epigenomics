---
name: dual-orientation-motif-scoring
description: Use when scanning centered, scored DNA sequences (e.g. FASTA with score
  headers) for transcription factor motif occurrences where motif strand is unknown,
  and positional enrichment profiles must reflect matches on either orientation.
when_to_use_negative:
- The assay resolves motif strand (e.g. ChIP-nexus) and only the forward orientation
  should be counted — use --orientations '+' instead of the dual-orientation max rule
- The goal is de novo motif discovery rather than scanning a fixed motif library,
  since MEPP currently only functions with a fixed motif library
- Sequences are not centered on a common anchor point (TSS, peak summit, or motif),
  so positional enrichment relative to an anchor is not meaningful
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0182
tools:
- name: MEPP
  role: Executes the PWM scanning over one-hot encoded sequences with the '+/-' dual-orientation
    max rule, MOODS-based thresholds, and margin-based smoothing of the motif score
    heatmap
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installs the MEPP package used to run the dual-orientation motif scanning
  repo: https://github.com/npdeloss/mepp
- name: HOMER
  role: Source of the motif matrices distributed with MEPP (data/homer.motifs.txt)
    that can be scanned in both orientations
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:b391fc90acaf80e80cf1cd6282e204e3ed0f16f0fe6ac8287ac91917337bded0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - HOMER
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0182
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# Dual-orientation motif scoring

## Summary

Scores every JASPAR PWM against both the forward and reverse-complement strands of one-hot encoded sequences and collapses the two scans with a max rule, so each positional heatmap cell carries the strongest strand-agnostic motif match excess (score − MOODS threshold).

## When to use

Use when scanning centered, scored DNA sequences (e.g. FASTA with score headers) for transcription factor motif occurrences where motif strand is unknown, and positional enrichment profiles must reflect matches on either orientation.

## When NOT to use

- The assay resolves motif strand (e.g. ChIP-nexus) and only the forward orientation should be counted — use --orientations '+' instead of the dual-orientation max rule
- The goal is de novo motif discovery rather than scanning a fixed motif library, since MEPP currently only functions with a fixed motif library
- Sequences are not centered on a common anchor point (TSS, peak summit, or motif), so positional enrichment relative to an anchor is not meaningful

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score'
- JASPAR-format motif matrices file (e.g. from JASPAR downloads or MEPP's data/homer.motifs.txt)
- Optional MOODS background composition as four floats (default 0.25 0.25 0.25 0.25)

## Outputs

- Motif-by-position excess-score heatmap matrix (one row per motif, columns spanning the padded sequence length)
- Positional enrichment profile plots (png/svg) with orientation-collapsed motif match brightness scaled by --sigma relative to nonzero match score standard deviation

## How to apply

One-hot encode the uniform-length padded sequences and scan each one with every JASPAR PWM (e.g. data/homer.motifs.txt or a JASPAR-formatted matrices file), recording at each position the excess score max(0, score − threshold). Set the PWM match threshold via MOODS using --pval 0.0001 and --pcount 0.0001 against the background composition (--bg, default 0.25 0.25 0.25 0.25). Apply the dual-orientation max rule: select --orientations '+/-' so each heatmap cell holds the higher excess score between the forward and reverse-complement scans; choose '+' alone only when the assay (e.g. ChIP-nexus) retains strand resolution. Optionally smooth the resulting heatmap rows with average pooling of window w = 1 + 2m, where m is the --margin (default 2), blurring motif matches to account for inexact positional positioning.

## Related tools

- **MEPP** (Executes the PWM scanning over one-hot encoded sequences with the '+/-' dual-orientation max rule, MOODS-based thresholds, and margin-based smoothing of the motif score heatmap) — https://github.com/npdeloss/mepp
- **pip** (Installs the MEPP package used to run the dual-orientation motif scanning) — https://github.com/npdeloss/mepp
- **HOMER** (Source of the motif matrices distributed with MEPP (data/homer.motifs.txt) that can be scanned in both orientations)

## Examples

```
mepp --fa scored.fa --motifs data/homer.motifs.txt --out mepp_out --orientations +/- --margin 2 --pval 0.0001
```

## Evaluation signals

- The output heatmap table has one row per motif and columns spanning the padded sequence length
- Each heatmap cell value equals max(0, forward excess, reverse-complement excess) and is 0 wherever neither strand exceeds the MOODS threshold (--pval 0.0001, --pcount 0.0001)
- Changing --orientations from '+/-' to '+' only decreases or leaves unchanged heatmap cell values (forward strand is a subset of the max rule)
- The pooled heatmap row length is consistent with the average pooling window w = 1 + 2m for the chosen --margin
- In strand-unresolved assays (e.g. Nanog ChIP-seq), the dual-orientation profile shows a sharp central motif peak at the peak summit rather than a strand-split signal

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- GPU memory usage from permutation testing (--perms, default 1000) and --gjobs crowding can cause profile generation to fail; use --nogpu for CPU-only parallel processing of motif jobs
- Sequences exceeding the --dgt degenerate-base percentage (default 100) are rejected, and the dual-orientation rule discards strand information that ChIP-nexus-type assays could otherwise exploit

## Evidence

- [readme] Dual-orientation max rule for heatmap cells: [evidence span withheld — anchor status: external_doc]
- [readme] MOODS threshold parameters: [evidence span withheld — anchor status: external_doc]
- [readme] Margin-based smoothing of motif matches: [evidence span withheld — anchor status: external_doc]
- [abstract] Positional profile relative to anchor points while controlling bias: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [methods] Installation via pip: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
