---
name: periodicity-detection-in-enrichment-profiles
description: Use when you have ChIP-seq-scored DNA sequences centered on an anchor
  motif (e.g. Nanog motif sites scored by mESC Nanog ChIP-seq signal) and want to
  test whether a second TF's motif (e.g. Sox2) occurs at periodic, helical-phase offsets
  rather than at a single fixed spacing.
when_to_use_negative:
- You only need motif presence/absence enrichment across a set of sequences without
  positional information — zero-or-one-occurrence-per-sequence MEA tools (HOMER, MEIRLOP)
  are the wrong granularity, and MEPP's positional profile is unnecessary.
- You need de novo motif discovery/assembly — the article notes 'MEPP currently functions
  with a fixed motif library'; de novo recognition is left to future work.
- Your sequences are not centered on a biologically meaningful anchor point (e.g.
  TSS or a motif), since the profile is defined relative to such anchor points.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: Computes the position-dependent motif enrichment profile relative to central
    anchor motifs, with permutation-based confidence intervals and multiple-testing-adjusted
    positional correlations used to detect ~10 bp helical periodicity.
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installation channel for MEPP per the methods instructions.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:bbbb009daa242c94ef0c3216b8f5a6e512623583ed3f88da39892e289be603ea
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
n_claims: 7
---

# periodicity-detection-in-enrichment-profiles

## Summary

Detects helical (~10 bp) spacing periodicity in positional motif enrichment by running MEPP on sequences centered on an anchor motif and inspecting the resulting positional enrichment profile for regularly spaced peaks flanking the anchor, as demonstrated for Sox2 motif enrichment around Nanog motif sites in mESCs.

## When to use

Use when you have ChIP-seq-scored DNA sequences centered on an anchor motif (e.g. Nanog motif sites scored by mESC Nanog ChIP-seq signal) and want to test whether a second TF's motif (e.g. Sox2) occurs at periodic, helical-phase offsets rather than at a single fixed spacing.

## When NOT to use

- You only need motif presence/absence enrichment across a set of sequences without positional information — zero-or-one-occurrence-per-sequence MEA tools (HOMER, MEIRLOP) are the wrong granularity, and MEPP's positional profile is unnecessary.
- You need de novo motif discovery/assembly — the article notes 'MEPP currently functions with a fixed motif library'; de novo recognition is left to future work.
- Your sequences are not centered on a biologically meaningful anchor point (e.g. TSS or a motif), since the profile is defined relative to such anchor points.

## Inputs

- Scored FASTA file of sequences centered on anchor motif sites, headers formatted '>sequence_name sequence_score' (sequences ±200 bp around deduplicated, RepeatMasker-filtered mm10 Nanog motif sites, scored by mESC Nanog ChIP-seq signal from GSE203135)
- Motif matrices file in JASPAR format (fixed motif library, e.g. JASPAR 2022, containing the query motif such as Sox2 and the anchor motif Nanog)

## Outputs

- MEPP positional enrichment plot (PNG and SVG, default 10x10 inches at 300 DPI) showing query motif enrichment as a function of position relative to the centered anchor motif
- Motif heatmaps and a clustering table with positional correlation p-values adjusted via statsmodels multiple testing (fdr_by, alpha 0.01)
- Output directory (--out) containing plots, confidence intervals from permutation testing, and motif match data

## How to apply

Extract ±200 bp sequences centered on deduplicated, RepeatMasker-filtered anchor motif sites (e.g. Nanog motifs from mm10) and score each sequence on a continuum using a ChIP-seq signal (e.g. Nanog mESC signal from GEO GSE203135); write them as a FASTA whose headers carry the score in the form '>sequence_name sequence_score'. Install MEPP via 'pip install mepp' (or 'pip install git+https://github.com/npdeloss/mepp@main') and run it with a fixed JASPAR 2022 motif library so the query motif's (e.g. Sox2's) positional enrichment is computed relative to the central anchor motifs. MEPP's score-based principle avoids arbitrary threshold selection and controls for lower-order nucleotide bias while profiling motif enrichment as a function of position rather than presence/absence (unlike ZOOPs-based MEA methods such as MEIRLOP and HOMER). Judge success by whether the positional profile shows repeating enrichment peaks with a ~10 bp period flanking the anchor (one DNA helical turn), matching the pattern reported for Sox2 around Nanog motifs; MEPP also emits confidence intervals from permutation testing (--perms 1000 default) and multiple-testing-adjusted positional correlations (--mtmethod fdr_by, --mtalpha 0.01) to support the call.

## Related tools

- **MEPP** (Computes the position-dependent motif enrichment profile relative to central anchor motifs, with permutation-based confidence intervals and multiple-testing-adjusted positional correlations used to detect ~10 bp helical periodicity.) — https://github.com/npdeloss/mepp
- **pip** (Installation channel for MEPP per the methods instructions.)

## Examples

```
pip install mepp; mepp --fa nanog_centered_scored.fa --motifs jaspar2022_motifs.txt --out nanog_sox2_mepp --perms 1000
```

## Evaluation signals

- Sox2 positional enrichment profile around central Nanog motifs shows periodic peaks with a ~10 bp period flanking the anchor, matching the reported Figure 4 pattern.
- Input FASTA headers follow MEPP's required format '>sequence_name sequence_score', with sequences centered on the anchor motif and scored on a continuum by ChIP-seq signal (no arbitrary score thresholds).
- Positional correlation p-values in the clustering table survive MEPP's thorough multiple testing (fdr_by at alpha 0.01).
- Permutation-based confidence intervals (--perms 1000) around the enrichment profile exclude baseline at the periodic peak positions.
- Sequences were deduplicated and RepeatMasker-filtered before extraction from mm10, and degenerate-sequence rejection (--dgt) is set appropriately for repeat control.

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work.
- No software version is specified for MEPP and no changelog exists for the repository.
- GPU memory usage can be significant during permutation testing (--perms 1000 default), and careless --gjobs settings can crowd jobs out of GPU memory causing profile generation to fail (mitigated by --attempts 10 retry logic).
- Detecting ~10 bp periodicity requires enough centered, scored motif sites; MEPP profiles relative to anchor points, so without a meaningful anchor (TSS or motif) the positional profile is undefined.

## Evidence

- [abstract] MEPP recovers ~10 bp helical Sox2 spacing around Nanog motifs: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [intro] score-based principle avoids thresholds and sequence bias: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [methods] install via pip: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [discussion] MEPP fixed motif library limitation: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs"
- [readme] README scored fasta input format: "--fa TEXT                       Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name"
- [readme] README JASPAR motif matrices requirement: "--motifs TEXT                   Path to a motif matrices file in JASPAR
                                      format. As a start, one can be obtained
                                      through the"
