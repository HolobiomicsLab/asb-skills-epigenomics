---
name: positional-enrichment-profile-visualization
description: Use when you have centered, scored genomic sequences (e.g., ±200 bp around
  ChIP-seq or ChIP-nexus peak summits with signal scores) and need to determine at
  which positions a TF binding motif is enriched relative to the sequence center.
when_to_use_negative:
- 'You need de novo motif discovery from the sequences: MEPP currently functions only
  with a fixed motif library; de novo motif recognition/assembly is left to future
  work'
- Your sequences are not centered on a biologically meaningful anchor point, so positional
  enrichment relative to the center would be uninterpretable
- You only need a single motif enrichment score without positional information (e.g.,
  standard MEA); a non-positional method suffices
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: Core tool that scores motif matches (via MOODS) in scored, centered sequences
    and renders positional enrichment profiles and heatmaps
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation method for MEPP into the execution environment
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:e0b200792ecec2bb33e1ac5e1620cdda8e5b8e71a27ea44cbfcb8a1dac09d9e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# positional-enrichment-profile-visualization

## Summary

Generate motif enrichment positional profiles (MEPP) that quantify where a transcription factor's motif occurs relative to anchor points (e.g., peak summits, TSSs) across scored genomic sequences, using a score-based continuum (e.g., MACS2 narrowpeak signal scores) instead of arbitrary thresholds.

## When to use

Use when you have centered, scored genomic sequences (e.g., ±200 bp around ChIP-seq or ChIP-nexus peak summits with signal scores) and need to determine at which positions a TF binding motif is enriched relative to the sequence center.

## When NOT to use

- You need de novo motif discovery from the sequences: MEPP currently functions only with a fixed motif library; de novo motif recognition/assembly is left to future work
- Your sequences are not centered on a biologically meaningful anchor point, so positional enrichment relative to the center would be uninterpretable
- You only need a single motif enrichment score without positional information (e.g., standard MEA); a non-positional method suffices

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (sequences centered on anchor points such as peak summits or TSSs)
- Motif matrices file in JASPAR format (e.g., Nanog motif from JASPAR, or data/homer.motifs.txt bundled with MEPP)
- Optional: signal scores from MACS2 narrowpeak files used as the sequence score continuum

## Outputs

- MEPP positional enrichment plots per motif (png and svg by default, 300 dpi)
- Motif heatmaps showing positional enrichment relative to the sequence center
- Clustering table of positional correlations with adjusted p-values (statsmodels multiple testing, default fdr_by, alpha 0.01)

## How to apply

Extract sequences (e.g., ±200 bp) centered on each anchor point and write a scored FASTA where headers carry the sequence score (e.g., MACS2 narrowpeak signal scores as a continuum of biological activity, avoiding arbitrary peak thresholds). Install MEPP via pip (pip install mepp or pip install git+https://github.com/npdeloss/mepp@main) and supply a JASPAR-format motif file (HOMER-derived motifs ship in data/homer.motifs.txt). Run mepp on the scored FASTA, choosing --orientations appropriately: scan both '+' and '-' orientations when the assay cannot resolve motif strand (e.g., ChIP-seq), or a single orientation when strand resolution is available (e.g., ChIP-nexus). MEPP scores motif matches with MOODS at default --pval 0.0001 and --pcount 0.0001, computes a position-dependent enrichment profile, and renders heatmaps and positional plots; use --center to set the plot anchor if sequences are not centered at midpoints.

## Related tools

- **MEPP** (Core tool that scores motif matches (via MOODS) in scored, centered sequences and renders positional enrichment profiles and heatmaps) — https://github.com/npdeloss/mepp
- **pip** (Installation method for MEPP into the execution environment) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa nanog_nexus_summits.fa --motifs nanog.motif --out mepp_out --orientations +/-
```

## Evaluation signals

- Profile exhibits a sharp positive peak centered directly on the plot center when motifs genuinely localize to the anchor point (e.g., Nanog motif on ChIP-nexus peak summits)
- Strand-resolvable datasets (ChIP-nexus) yield sharp central motif peaks with resolvable flanking peaks (e.g., Oct4-Sox2-TCF-Nanoc twin peaks), whereas non-strand-resolved data (ChIP-seq) give broader, lower-amplitude profiles
- Scored FASTA headers parse correctly as '>sequence_name sequence_score'; sequences rejected if degenerate nucleotide content exceeds --dgt percentage
- Output directory contains both png and svg MEPP plots at 300 dpi, plus the clustering table with fdr_by-adjusted p-values at alpha 0.01
- Permutation-based confidence intervals (default 1000 permutations) bracket the expected enrichment at the center position

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified for MEPP and the repository provides no changelog
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail
- Motif match thresholds depend on the supplied background DNA composition via --bg (default 0.25 0.25 0.25 0.25); skewed sequence composition can mis-set thresholds

## Evidence

- [abstract] Score-based motif enrichment analysis (MEA) is typically applied to regulatory DNA: "Score-based motif enrichment analysis (MEA) is typically applied to regulatory DNA"
- [methods] pip installation: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [discussion] positional relevance demonstration: "We have demonstrated MEPP as a novel means of quantifying and visualizing the positional relevance of a motif across multiple centered genomic sequences."
- [discussion] fixed motif library limitation: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs"
- [readme] scored fasta header format: "Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name
                                      sequence_score"."
- [readme] orientations option: "--orientations TEXT             Comma-separated list of motif orientations
                                      to analyze for CPU multiprocessing. Values
                                      in"
