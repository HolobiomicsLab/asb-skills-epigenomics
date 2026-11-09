---
name: chip-seq-signal-scoring-of-motif-sites
description: Use when motif-centered genomic sequences need a continuous biological-activity
  score (e.g. mESC Nanog ChIP-seq signal from GSE203135) so MEPP can profile partner-motif
  enrichment along a score continuum instead of arbitrary binding thresholds.
when_to_use_negative:
- No continuous activity track exists for the motif sites (no ChIP-seq signal for
  the factor) — there is nothing to rank sequences on and a score-based analysis cannot
  proceed
- 'The goal is de novo motif discovery: MEPP currently functions only with a fixed
  motif library, so scoring sequences will not reveal unmodeled partner motifs'
- The design already bins sites into discrete classes or quartiles (CentriMo/TFEA-style
  analyses) rather than scoring each sequence on a continuum
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0160
tools:
- name: MEPP
  role: Consumes the ChIP-seq-scored FASTA (--fa, headers '>sequence_name sequence_score')
    and computes the position-dependent partner-motif (Sox2) enrichment profile around
    central Nanog anchors while controlling for sequence bias
  repo: https://github.com/npdeloss/mepp
- name: MOODS
  role: Sets motif match thresholds (--pval 0.0001, --pcount 0.0001) when scoring
    motif occurrences within the signal-scored sequences during the MEPP run
- name: pip
  role: Installs MEPP before scoring/profiling, via 'pip install mepp' or 'pip install
    git+https://github.com/npdeloss/mepp@main'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:50a3def17e619d9b2e9dd5e09439e6a6a2c639ed1e4fae7706bd650c20e31936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - MOODS
  - pip
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# chip-seq-signal-scoring-of-motif-sites

## Summary

Attaches a continuous ChIP-seq activity score to each deduplicated, RepeatMasker-filtered motif-centered sequence (e.g. ±200 bp around mm10 Nanog motif sites, scored with mESC Nanog ChIP-seq signal from GEO GSE203135) and writes MEPP's required '>sequence_name sequence_score' FASTA headers. The continuum scoring is what lets MEPP profile Sox2 motif enrichment positionally around central Nanog anchors without arbitrary binding thresholds.

## When to use

Use when motif-centered genomic sequences need a continuous biological-activity score (e.g. mESC Nanog ChIP-seq signal from GSE203135) so MEPP can profile partner-motif enrichment along a score continuum instead of arbitrary binding thresholds.

## When NOT to use

- No continuous activity track exists for the motif sites (no ChIP-seq signal for the factor) — there is nothing to rank sequences on and a score-based analysis cannot proceed
- The goal is de novo motif discovery: MEPP currently functions only with a fixed motif library, so scoring sequences will not reveal unmodeled partner motifs
- The design already bins sites into discrete classes or quartiles (CentriMo/TFEA-style analyses) rather than scoring each sequence on a continuum

## Inputs

- Deduplicated, RepeatMasker-filtered motif-site coordinates (e.g. Nanog motif sites in mm10) used as anchors for ±200 bp sequence extraction
- mESC Nanog ChIP-seq signal track from GEO accession GSE203135 (bigWig, per the MEPP motif-centered scored-analysis walkthrough) used to assign per-site scores
- Scored FASTA of centered sequences with headers in MEPP's '>sequence_name sequence_score' format (supplied via MEPP --fa)
- JASPAR-format motif matrices file (e.g. the fixed JASPAR 2022 library containing the Sox2 motif) supplied via MEPP --motifs

## Outputs

- Scored FASTA file: one continuous ChIP-seq signal score per motif-centered sequence, encoded in the header
- MEPP positional enrichment profile plots (png/svg) of Sox2 motif enrichment versus distance from central Nanog motifs
- MEPP clustering table of positionally correlated motifs with multiple-testing-adjusted p-values

## How to apply

Extract ±200 bp sequences centered on deduplicated, RepeatMasker-filtered anchor motif sites (e.g. Nanog motifs from the mm10 assembly) and read the mESC Nanog ChIP-seq (bigWig) signal from GSE203135 at each center, giving every site a continuous score rather than a bound/unbound call. Write the sequences to a scored FASTA whose headers follow MEPP's required '>sequence_name sequence_score' format, since this score is what MEPP uses to order sequences along the enrichment continuum. Keep scores on the continuum — the score-based principle avoids arbitrary threshold selection while controlling for lower-order nucleotide sequence bias — and optionally lower MEPP's --dgt flag below its default of 100 to reject sequences with too many degenerate (non-ACGT) bases left over from repeats. Run MEPP against the fixed JASPAR 2022 motif library (--motifs) with default MOODS thresholds (--pval 0.0001, --pcount 0.0001) to profile partner-motif (Sox2) enrichment as a function of position relative to the central Nanog anchor. Judge success by a Sox2 positional profile showing periodic peaks spaced ~10 bp on either side of the central Nanog motifs, the helical binding signature reported in Figure 4.

## Related tools

- **MEPP** (Consumes the ChIP-seq-scored FASTA (--fa, headers '>sequence_name sequence_score') and computes the position-dependent partner-motif (Sox2) enrichment profile around central Nanog anchors while controlling for sequence bias) — https://github.com/npdeloss/mepp
- **MOODS** (Sets motif match thresholds (--pval 0.0001, --pcount 0.0001) when scoring motif occurrences within the signal-scored sequences during the MEPP run)
- **pip** (Installs MEPP before scoring/profiling, via 'pip install mepp' or 'pip install git+https://github.com/npdeloss/mepp@main')

## Examples

```
mepp --fa scored_nanog_sites.fa --motifs JASPAR2022_motifs.jaspar --out sox2_nanog_mepp
```

## Evaluation signals

- Every FASTA header parses as '>sequence_name <numeric score>' and MEPP --fa accepts the file without header-parsing errors
- Score distribution is continuous — no bound/unbound threshold or quartile binning was applied — consistent with the score-based principle
- Sequences exceeding the --dgt degenerate-base percentage were rejected (default 100 means no rejection, so verify the chosen value matches the repeat-filtering intent)
- The resulting Sox2 positional profile shows peaks recurring at ~10 bp spacing flanking the central Nanog motifs (helical periodicity, cf. Figure 4)
- Spot-check several motif centers against the GSE203135 bigWig: header scores match the extracted ChIP-seq signal values

## Limitations

- MEPP only functions with a fixed motif library; de novo motif recognition/assembly is left to future work, so unmodeled partner motifs cannot surface regardless of scoring
- Sequences with many degenerate bases are rejected by --dgt; heavy repeat content can silently shrink the analyzed site set if the flag is not set appropriately
- Permutation testing (--perms, default 1000) can consume substantial GPU memory, and multiple gjobs risk crowding each other out of GPU memory, causing profile generation to fail

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs): "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs)"
- [intro] The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias.: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: [evidence span withheld — anchor status: external_doc]
- [readme] Percentage of sequence that can be degenerate (Not A, C, G, or T) before being rejected from the analysis. Useful for filtering out repeats.: [evidence span withheld — anchor status: external_doc]
- [supplementary] All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135: "All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135"
- [discussion] MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work"
