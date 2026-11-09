---
name: repeatmasker-transposable-element-annotation-processing
description: Use when a strand-asymmetry or breakpoint-enrichment analysis needs class-filtered,
  strand-annotated transposable element coordinates from UCSC hg38 rmsk (e.g. LINE/SINE/LTR)
  to orient mutations or assign background orientation probabilities.
when_to_use_negative:
- The query features already carry strand annotations — the orientation step against
  repeats is unnecessary; feed them directly into Asymmetron's asymmetry estimation
  functions.
- The analysis targets a non-hg38 genome or a non-mammalian species for which the
  11 March 2019 UCSC rmsk table is not the correct coordinate system.
- The question concerns palindromic sequence content or nucleotide compositional skews
  rather than orienting features relative to strand-annotated transposable elements.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0102
tools:
- name: Asymmetron
  role: Provides the orientation function that performs strand assignment of an unassigned
    feature (e.g. SV breakpoint) based on the overlapping strand-annotated repeat
    element
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Backend for Asymmetron's core BED-formatted file comparison, used to intersect
    breakpoints with filtered repeat intervals
- name: bedtools
  role: Underlying toolkit (e.g. 'bedtools closest' logic) used to determine the closest
    overlapping repeat element when ties must be resolved
- name: pandas
  role: Used in Python for loading and class-filtering the rmsk repeat annotation
    table and for downstream statistical analysis of per-class counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:48e9d02a63464a60863e34eb7168768c58ec72f54241bf85bfa00dc8812a6746
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
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# repeatmasker-transposable-element-annotation-processing

## Summary

Processes UCSC hg38 RepeatMasker (rmsk) repeat annotations into strand-annotated, class-filtered transposable element intervals (LINE, SINE, LTR) that serve as the reference elements for orienting unassigned genomic features such as gnomAD germline structural variant breakpoints in strand-asymmetry analyses with Asymmetron.

## When to use

Use when a strand-asymmetry or breakpoint-enrichment analysis needs class-filtered, strand-annotated transposable element coordinates from UCSC hg38 rmsk (e.g. LINE/SINE/LTR) to orient mutations or assign background orientation probabilities.

## When NOT to use

- The query features already carry strand annotations — the orientation step against repeats is unnecessary; feed them directly into Asymmetron's asymmetry estimation functions.
- The analysis targets a non-hg38 genome or a non-mammalian species for which the 11 March 2019 UCSC rmsk table is not the correct coordinate system.
- The question concerns palindromic sequence content or nucleotide compositional skews rather than orienting features relative to strand-annotated transposable elements.

## Inputs

- UCSC hg38 rmsk.txt.gz RepeatMasker table (version from 11 March 2019) from hgdownload.soe.ucsc.edu
- Optional: gnomAD v2 germline structural variant breakpoints (unassigned strand) to orient against the repeats
- GENCODE v33 protein-coding gene annotation (for downstream template/non-template assignment)
- ENCODE Repli-seq BG02ES (Release 2) data lifted to hg38 (for downstream leading/lagging assignment)

## Outputs

- BED-formatted transposable element intervals restricted to LINE/SINE (and optionally LTR) classes, with '+'/'-' strand annotations
- Per-element-class background strand-orientation probabilities (non-template/leading over total occurrences) usable for adjusted asymmetry ratios
- Strand-annotated breakpoints (oriented_breakpoints.bed) when used with the Asymmetron orientation function

## How to apply

Download the RepeatMasker table from UCSC goldenPath for hg38 (rmsk.txt.gz, 11 March 2019 version) and parse the repeat coordinates into BED-formatted intervals retaining the strand column. Filter the annotation to the transposable element classes required by the analysis — the article's pipeline filters to LINE, SINE and LTR elements (the germline SV task further restricts to LINE and SINE). Because the Asymmetron BED-comparison core runs on Pybedtools, keep intervals in valid 6+ column BED format with '+'/'-' strand values. When an overlapping query feature (e.g. an unassigned breakpoint) intersects multiple repeat elements, the Asymmetron orientation function keeps the element with the minimal center-to-center distance |(s+e)/2 - (s'+e')/2|. The filtered, strand-annotated repeats then support both orientation of third features and computation of background transcriptional/replicative asymmetry probabilities per element class (template/non-template via GENCODE v33 protein-coding genes; leading/lagging via ENCODE BG02ES Repli-seq).

## Related tools

- **Asymmetron** (Provides the orientation function that performs strand assignment of an unassigned feature (e.g. SV breakpoint) based on the overlapping strand-annotated repeat element) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Backend for Asymmetron's core BED-formatted file comparison, used to intersect breakpoints with filtered repeat intervals)
- **bedtools** (Underlying toolkit (e.g. 'bedtools closest' logic) used to determine the closest overlapping repeat element when ties must be resolved)
- **pandas** (Used in Python for loading and class-filtering the rmsk repeat annotation table and for downstream statistical analysis of per-class counts)

## Examples

```
repeats = pd.read_csv('rmsk.txt.gz', sep='\t'); line_sine = repeats[repeats.repClass.isin(['LINE','SINE'])]
```

## Evaluation signals

- The filtered output contains only LINE and SINE (and optionally LTR) repClass entries, with all other repeat classes (DNA, Simple_repeat, etc.) removed.
- Every retained element carries a valid '+' or '-' strand value inherited from the rmsk table, with no missing strand fields.
- Coordinates reproduce the UCSC hg38 rmsk 11 March 2019 release when spot-checked against goldenPath records.
- When a breakpoint overlaps multiple repeats, the assigned element is the one with the minimal center-to-center distance |(s+e)/2 - (s'+e')/2|.
- Downstream, per-class background asymmetry ratios can be computed for both LINE and SINE and divided into observed breakpoint ratios to yield adjusted ratios with Bonferroni-corrected binomial P-values in germline_sv_strand_asymmetries.tsv.

## Limitations

- Analysis is version-sensitive: results depend on the specific rmsk release (11 March 2019) and genome build (hg38); other releases may shift element coordinates and orientation counts.
- Breakpoints overlapping several repeat elements are assigned by a center-to-center distance tie-break, which is a heuristic that may misassign orientation in dense repeat clusters.
- The article notes weaker replicative versus transcriptional asymmetry biases, so class-level background probabilities derived from these annotations may yield less significant replication-strand results (LTRs showed no significant correlation).

## Evidence

- [methods] Coordinates of transposable elements were derived for the human genome (hg38) from the http://hgdownload.soe.ucsc.edu/goldenPath/hg38/database/rmsk.txt.gz (version from 11 March 2019): "Coordinates of transposable elements were derived for the human genome (hg38) from the http://hgdownload.soe.ucsc.edu/goldenPath/hg38/database/rmsk.txt.gz (version from 11 March 2019)"
- [methods] and were filtered to include LINE, SINE and LTR transposable elements.: "and were filtered to include LINE, SINE and LTR transposable elements."
- [methods] A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [readme] Asymmetron can assign strand orientation to third features such as mutations, by orienting them relative to other genomic elements.: [evidence span withheld — anchor status: external_doc]
