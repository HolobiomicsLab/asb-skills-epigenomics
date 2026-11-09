---
name: structural-variant-breakpoint-analysis
description: Use when you have unstranded SV breakpoint coordinates (BED) plus strand-annotated
  repeat elements and need to test whether breakpoints show transcriptional or replicative
  strand biases.
when_to_use_negative:
- Breakpoints or genomic features already carry reliable strand annotation — the orientation-assignment
  function is only needed for unassigned features
- Analysis targets strand asymmetries within consecutive occurrences of a single strand-annotated
  element or between pairs of strand-assigned motifs, rather than orienting third-party
  features like mutations relative to overlapping elements
- The repeat annotation does not distinguish element orientation (e.g., non-directional
  repeat classes), since background correction requires element-class-specific orientation
  probabilities
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Asymmetron
  role: Python toolkit providing the orientation function that assigns '+'/'-' strand
    to unassigned breakpoints based on overlapping LINE/SINE elements, plus two-tailed
    binomial testing and Bonferroni correction
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted file comparison functions used for
    overlap-based orientation assignment
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: bedtools
  role: Provides the 'bedtools closest' function to determine the closest element
    when multiple elements overlap
- name: scipy
  role: Computes the two-tailed binomial test P-values with a custom background probability
    per element class
- name: pandas
  role: Statistical analysis and assembly of the per-class asymmetry ratio table
- name: numpy
  role: Numerical computation of observed/background asymmetry ratios
- name: matplotlib
  role: Visualization of strand asymmetry results
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:d110dea11a642a9c2897014f0bceb082bf723add479322b5ace55817fd0a77cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# structural-variant-breakpoint-analysis

## Summary

Orients unstranded germline structural variant breakpoints relative to transposable elements (LINEs/SINEs) using Asymmetron's orientation function, then detects transcriptional (template/non-template, via GENCODE v33) and replicative (leading/lagging, via ENCODE BG02ES Repli-seq) strand asymmetries after background correction and Bonferroni-adjusted binomial testing.

## When to use

Use when you have unstranded SV breakpoint coordinates (BED) plus strand-annotated repeat elements and need to test whether breakpoints show transcriptional or replicative strand biases.

## When NOT to use

- Breakpoints or genomic features already carry reliable strand annotation — the orientation-assignment function is only needed for unassigned features
- Analysis targets strand asymmetries within consecutive occurrences of a single strand-annotated element or between pairs of strand-assigned motifs, rather than orienting third-party features like mutations relative to overlapping elements
- The repeat annotation does not distinguish element orientation (e.g., non-directional repeat classes), since background correction requires element-class-specific orientation probabilities

## Inputs

- gnomAD v2 germline structural variant breakpoint coordinates (BED-formatted, unstranded)
- UCSC hg38 rmsk (RepeatMasker) table, 11 March 2019 version, filtered to LINE and SINE elements
- GENCODE v33 annotation filtered to protein-coding genes (template/non-template assignment)
- ENCODE Repli-seq BG02ES (Release 2) data lifted over to hg38 (leading/lagging orientation)

## Outputs

- germline_sv_strand_asymmetries.tsv: adjusted transcriptional and replicative strand asymmetry ratios with Bonferroni-corrected P-values per element class (LINE, SINE)
- oriented_breakpoints.bed: strand-annotated breakpoints with '+' or '-' assignments from overlapping elements

## How to apply

Load gnomAD v2 germline SV breakpoints and the UCSC hg38 rmsk table (11 March 2019 version), filtering the repeat annotation to LINE and SINE elements. Use the Asymmetron orientation function (built on Pybedtools) to assign '+' or '-' strand to each unassigned breakpoint based on the overlapping LINE/SINE element, keeping the element with the minimal center-to-center distance |(s+e)/2 - (s'+e')/2| when multiple elements overlap. Compute background transcriptional and replicative strand asymmetry ratios of the LINE/SINE elements themselves — non-template orientation assigned from GENCODE v33 protein-coding annotation, leading/lagging from ENCODE BG02ES Repli-seq-derived replication direction. Compute observed ratios for the oriented breakpoints, then derive adjusted asymmetry ratios by dividing observed by background; compute two-tailed binomial P-values with scipy, replacing the default binomial probability P with each element class's background asymmetry probability, and apply Bonferroni correction. The rationale is that a strand imbalance in breakpoints is only meaningful after correcting for the elements' own orientation preferences relative to transcription and replication.

## Related tools

- **Asymmetron** (Python toolkit providing the orientation function that assigns '+'/'-' strand to unassigned breakpoints based on overlapping LINE/SINE elements, plus two-tailed binomial testing and Bonferroni correction) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies Asymmetron's core BED-formatted file comparison functions used for overlap-based orientation assignment) — https://github.com/Ahituv-lab/Asymmetron
- **bedtools** (Provides the 'bedtools closest' function to determine the closest element when multiple elements overlap)
- **scipy** (Computes the two-tailed binomial test P-values with a custom background probability per element class)
- **pandas** (Statistical analysis and assembly of the per-class asymmetry ratio table)
- **numpy** (Numerical computation of observed/background asymmetry ratios)
- **matplotlib** (Visualization of strand asymmetry results)

## Examples

```
git clone https://github.com/Ahituv-lab/Asymmetron && conda activate asymmetron  # then run Asymmetron's orientation function on gnomAD v2 SV breakpoints against filtered hg38 rmsk LINE/SINE elements
```

## Evaluation signals

- Output table germline_sv_strand_asymmetries.tsv contains both adjusted transcriptional and replicative asymmetry ratios plus Bonferroni-corrected P-values for each element class (LINE and SINE)
- oriented_breakpoints.bed has every breakpoint assigned '+' or '-' consistent with the overlapping LINE/SINE element chosen by minimal center-to-center distance
- Adjusted ratios should be near or below 0.5 for template-strand preference; paper reports LINE and SINE associated with the template strand after background correction
- Unadjusted (observed) ratios should differ from adjusted ratios, confirming background element orientation was actually corrected for
- Bonferroni-corrected P-values should be reproducible when recomputed with scipy using each class's background asymmetry probability as the binomial P parameter

## Limitations

- Observed strand asymmetries are weaker for replication than transcription; dynamic changes in replication directionality during evolution are proposed as an explanation, and replication direction inferred from one cell line (BG02ES) may not generalize
- Whether the observed SV asymmetries at transposable elements are aggravated in cancer and other disorders, or associated with disease development, remains unknown
- Orientation assignment depends on overlapping repeat elements; breakpoints outside annotated LINE/SINE elements cannot be oriented by this method

## Evidence

- [methods] A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [methods] Background correction and binomial testing: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] Multiple-testing handling: "and are adjusted with Bonferroni correction in case of multiple tests."
- [intro] Demonstrated application to SV breakpoints: "Using Asymmetron we show that germline structural variant breakpoints can be oriented relative to transposable elements and find transcriptional and replicative strand asymmetries"
- [results] Background element-class preferences found: "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [readme] Tooling and dependencies: "conda create --name asymmetron pybedtools python=3.5 seaborn numpy"
