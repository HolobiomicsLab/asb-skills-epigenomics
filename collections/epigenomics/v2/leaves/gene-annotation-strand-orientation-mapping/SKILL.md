---
name: gene-annotation-strand-orientation-mapping
description: Use when you have strand-annotated BED files of motifs (e.g. TFBSs) and
  gene regions and need to test whether motif orientation is biased relative to transcriptional
  direction within transcribed regions.
when_to_use_negative:
- The goal is consecutive-occurrence asymmetry of a single feature, or pairwise asymmetry
  between two motifs in proximity — use Asymmetron's consecutive_patterns or pairwise_asymmetries
  instead of contained_asymmetries.
- Motifs or regions lack strand annotation and cannot be oriented — the function requires
  strand-assigned inputs (use the separate orientation function to assign strands
  first).
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function that classifies motif/region overlaps
    as same or opposite strand orientation and computes strand asymmetry statistics
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted file comparison / interval overlap
    for classifying motif-orientation pairs
- name: scipy
  role: Computes two-tailed binomial P-values for each transcription factor's strand
    asymmetry ratio
- name: pandas
  role: Used with numpy for statistical analysis/tabulation of asymmetry scores and
    P-values
- name: numpy
  role: Used with pandas and scipy for the statistical analysis of strand asymmetries
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:02bd4139a7c9cf7581e63576211df1a9323a8bc5dbea985c409418ff36c70341
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gene-annotation-strand-orientation-mapping

## Summary

Maps strand-annotated motifs (e.g. TFBSs) onto strand-annotated gene regions (GENCODE v33, hg38) using Asymmetron's contained_asymmetries function to compute transcriptional strand asymmetry ratios and detect non-random motif orientation relative to transcription direction.

## When to use

Use when you have strand-annotated BED files of motifs (e.g. TFBSs) and gene regions and need to test whether motif orientation is biased relative to transcriptional direction within transcribed regions.

## When NOT to use

- The goal is consecutive-occurrence asymmetry of a single feature, or pairwise asymmetry between two motifs in proximity — use Asymmetron's consecutive_patterns or pairwise_asymmetries instead of contained_asymmetries.
- Motifs or regions lack strand annotation and cannot be oriented — the function requires strand-assigned inputs (use the separate orientation function to assign strands first).

## Inputs

- Strand-annotated BED file of protein-coding gene regions from GENCODE v33 (hg38)
- Strand-annotated BED file of TFBS/motif occurrences (e.g. FIMO-scanned JASPAR 2020 CORE vertebrates non-redundant PWMs, or Unibind PWM motif maps 2019 release)
- Human genome build hg38 for motif scanning background nucleotide frequencies

## Outputs

- Per-transcription-factor transcriptional strand asymmetry ratio (non-template / (template + non-template) occurrences)
- Two-tailed binomial P-value per transcription factor (scipy, expected P = 0.5)
- Bonferroni-corrected P-value per transcription factor
- Fraction of transcription factor PWMs with significant transcriptional strand bias (expected ~45% of 551 PWMs)

## How to apply

Prepare a strand-annotated BED file of protein-coding gene regions from GENCODE v33 (hg38) and a strand-annotated BED file of motif occurrences (e.g. TFBSs scanned with FIMO at P-value <10−6, or Unibind PWM motif maps). Run Asymmetron's contained_asymmetries function with the gene regions as regions and motifs as motifs; it uses Pybedtools-based interval overlap to classify each overlapping motif/region pair as same or opposite strand orientation with expected_asym = 0.5. Compute the transcriptional strand asymmetry ratio per motif source (e.g. per transcription factor PWM, out of 551 JASPAR 2020 CORE non-redundant PWMs) as non-template-strand occurrences divided by template plus non-template occurrences. Calculate two-tailed binomial P-values in scipy assuming P = 0.5, then apply Bonferroni correction across all tested transcription factors. Tabulate the asymmetry score, P-value, and corrected P-value per transcription factor, and report the fraction with significant bias.

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function that classifies motif/region overlaps as same or opposite strand orientation and computes strand asymmetry statistics) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies Asymmetron's core BED-formatted file comparison / interval overlap for classifying motif-orientation pairs)
- **scipy** (Computes two-tailed binomial P-values for each transcription factor's strand asymmetry ratio)
- **pandas** (Used with numpy for statistical analysis/tabulation of asymmetry scores and P-values)
- **numpy** (Used with pandas and scipy for the statistical analysis of strand asymmetries)

## Examples

```
conda create --name asymmetron pybedtools python=3.5 seaborn numpy && conda activate asymmetron && git clone https://github.com/Ahituv-lab/Asymmetron
```

## Evaluation signals

- The reproducible fraction of transcription factors with significant Bonferroni-corrected strand bias matches the reported 45% (248 of 551 PWMs at P-value < 0.05, Bonferroni corrected).
- Every motif/region overlap pair is classified as same- or opposite-strand with expected_asym = 0.5 under the null; significant deviations should correspond to ratio departures from 0.5.
- PWMs with no FIMO matches meeting the P-value <10−6 threshold are filtered out before testing (resulting in 551 PWMs from JASPAR 2020 CORE non-redundant).
- All gene regions are protein-coding only (GENCODE v33 filtered) and coordinates are on hg38.
- Output table contains one row per transcription factor with asymmetry score, raw binomial P-value, and Bonferroni-corrected P-value columns.

## Limitations

- Requires strand annotation on both motifs and regions; unannotated features must first be oriented via Asymmetron's strand-assignment function based on an overlapping feature.
- Significant orientation bias does not establish mechanism — preferential affinity for forward vs reverse-complement orientation and interaction/interference with RNA polymerase is proposed but not established.
- Motif scanning depends on a FIMO P-value threshold (P <10−6) using human genome nucleotide frequencies as background; PWMs with no matches meeting the threshold are excluded.

## Evidence

- [abstract] Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features.: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature"
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] requiring a minimum P-value <10−6 (29).: "requiring a minimum P-value <10−6 (29)."
- [results] We filtered out transcription factor Position Weight Matrices (PWMs) for which there were no matches meeting the P-value threshold, resulting in 551 PWMs: "We filtered out transcription factor Position Weight Matrices (PWMs) for which there were no matches meeting the P-value threshold, resulting in 551 PWMs"
- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions"
