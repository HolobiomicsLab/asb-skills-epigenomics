---
name: repeatmasker-repetitive-element-filtering
description: Use when preparing centered, score-carrying motif-site sequences (e.g.
  ±200 bp around Nanog ChIP-seq motif sites) for positional motif enrichment, and
  repeats must be removed to avoid confounding motif-matching with low-complexity
  DNA.
when_to_use_negative:
- The goal is de novo motif discovery — MEPP only functions with a fixed motif library,
  so repeat filtering in its pipeline cannot assemble new motifs.
- Sequences are already filtered (e.g. an existing deduplicated, RepeatMasker-masked
  site set) — re-filtering adds no information and risks discarding valid sites.
- Input is not centered, scored sequences (e.g. raw FASTQ or an unsorted BAM) — this
  skill presumes motif sites already called, deduplicated, and score-ordered for MEPP.
edam_operation: http://edamontology.org/operation_0368
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: RepeatMasker
  role: Provides repetitive-element annotation used to exclude repeat-overlapping
    motif sites before sequence extraction and MEPP scoring.
- name: MEPP
  role: Consumes the filtered scored FASTA; its --dgt parameter performs a final degenerate-sequence
    rejection step to keep repeat/low-complexity sequence out of the positional enrichment
    profile.
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:e175978d5cc26d5018cc509e3b46d946ae2d3ca0b214bee27b767425e8e0d3ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - RepeatMasker
  - pip
  edam_operation: http://edamontology.org/operation_0368
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# repeatmasker-repetitive-element-filtering

## Summary

Filters repetitive/low-complexity sequence out of ChIP-seq-derived motif site sets (e.g. deduplicated, RepeatMasker-filtered Nanog motif sites in mm10) so that downstream MEPP positional enrichment profiles reflect specific TF binding rather than repeat-driven spurious motif matches.

## When to use

Use when preparing centered, score-carrying motif-site sequences (e.g. ±200 bp around Nanog ChIP-seq motif sites) for positional motif enrichment, and repeats must be removed to avoid confounding motif-matching with low-complexity DNA.

## When NOT to use

- The goal is de novo motif discovery — MEPP only functions with a fixed motif library, so repeat filtering in its pipeline cannot assemble new motifs.
- Sequences are already filtered (e.g. an existing deduplicated, RepeatMasker-masked site set) — re-filtering adds no information and risks discarding valid sites.
- Input is not centered, scored sequences (e.g. raw FASTQ or an unsorted BAM) — this skill presumes motif sites already called, deduplicated, and score-ordered for MEPP.

## Inputs

- Deduplicated Nanog motif-site coordinates from mm10 with per-site ChIP-seq scores (GSE203135)
- RepeatMasker annotation for the mm10 genome assembly
- Centered ±200 bp sequences in scored FASTA format (headers '>sequence_name sequence_score')
- JASPAR 2022 motif matrices file in JASPAR format

## Outputs

- RepeatMasker-filtered, degenerate-free scored FASTA used as MEPP input
- MEPP positional enrichment profile plots (png/svg) of Sox2 motif enrichment relative to central Nanog motif anchors
- MEPP clustering table with position-correlation p-values (fdr_by adjusted)

## How to apply

Starting from deduplicated, score-ordered motif sites (Nanog motif sites scored on a continuum by mESC ChIP-seq signal, GEO GSE203135, mm10), remove entries overlapping repetitive elements (RepeatMasker annotation) before sequence extraction. Because MEPP itself reads scored FASTA, apply a degenerate-base guard at run time: the --dgt option rejects any sequence whose percentage of non-ACGT characters exceeds a set threshold, which the README describes as 'useful for filtering out repeats'; lower --dgt from its permissive default of 100 to a strict value to exclude repeat-derived or low-complexity sequences. This filtering preserves the MEPP design principle of controlling for sequence bias while avoiding arbitrary threshold selection on the biological score axis. Correct application is judged by confirming no RepeatMasker-overlapping or degenerate-rich sequences survive, and that the resulting Sox2 positional profile around the central Nanog motif anchors shows the expected ~10 bp helical periodicity rather than artifacts concentrated in repeat-rich sequence.

## Related tools

- **RepeatMasker** (Provides repetitive-element annotation used to exclude repeat-overlapping motif sites before sequence extraction and MEPP scoring.)
- **MEPP** (Consumes the filtered scored FASTA; its --dgt parameter performs a final degenerate-sequence rejection step to keep repeat/low-complexity sequence out of the positional enrichment profile.) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa nanog_sites_scored.fa --motifs jaspar2022.txt --out mepp_out --dgt 25
```

## Evaluation signals

- No retained sequence overlaps RepeatMasker-annotated repetitive elements and site counts drop accordingly after filtering relative to the raw deduplicated set.
- MEPP does not reject additional sequences for degeneracy: with a strict --dgt threshold, the fraction of rejected sequences should be near zero if pre-filtering worked.
- The resulting Sox2 positional enrichment profile exhibits periodic peaks with a ~10 bp period flanking the central Nanog motifs, matching the published Figure 4 pattern.
- Profile confidence intervals from the default 1000 permutations and thorough multiple testing (fdr_by, alpha 0.01) do not flag repeat-correlated motifs as enriched at anchor positions.
- Score-based enrichment on the filtered set still spans the continuum of ChIP-seq scores, confirming filtering did not bias the score distribution by arbitrarily thresholding.

## Limitations

- MEPP's --dgt guard is a sequence-composition heuristic (percentage of non-ACGT bases); it does not itself perform RepeatMasker-class repeat annotation, so explicit RepeatMasker filtering must be done upstream.
- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work, so filtered inputs cannot compensate for missing motifs.
- The --dgt default is 100 (no rejection); leaving it unfiltered silently admits repeat-derived sequences into the analysis.

## Evidence

- [readme] --dgt INTEGER Percentage of sequence that can be degenerate (Not A, C, G, or T) before being rejected from the analysis. Useful for filtering out repeats. Default: 100: [evidence span withheld — anchor status: external_doc]
- [intro] The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias.: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences"
