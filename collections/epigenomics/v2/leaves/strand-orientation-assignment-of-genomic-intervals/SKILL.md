---
name: strand-orientation-assignment-of-genomic-intervals
description: Use when a BED/interval file lacks strand annotation and downstream strand-asymmetry
  analysis requires each feature to be oriented relative to an overlapping strand-assigned
  genomic element.
when_to_use_negative:
- Input features already carry strand annotation — use Asymmetron's consecutive/contained/pairwise
  asymmetry functions directly instead
- Features do not overlap any strand-annotated element in the reference set, so orientation
  cannot be inferred from overlap
- The analysis goal is palindromic-sequence orientation or de novo motif discovery
  rather than orienting intervals against annotated elements
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Asymmetron
  role: Provides the orientation function that assigns '+'/'-' strand to unannotated
    features based on overlapping strand-assigned elements
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Implements the core BED-formatted file comparison underlying the overlap/closest-element
    selection
- name: bedtools
  role: Underlying 'bedtools closest' logic used by Asymmetron to determine the closest
    element when multiple candidates overlap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:e42149905a11cbe3a1acd126b1498123dd6e8297ee2647a4c0a39c6672044f72
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# strand-orientation-assignment-of-genomic-intervals

## Summary

Assigns a '+' or '-' strand to strand-unannotated genomic features (e.g. gnomAD v2 germline structural variant breakpoints) using Asymmetron's orientation function, which anchors each feature to an overlapping strand-annotated element such as LINE/SINE records from the UCSC hg38 rmsk table.

## When to use

Use when a BED/interval file lacks strand annotation and downstream strand-asymmetry analysis requires each feature to be oriented relative to an overlapping strand-assigned genomic element.

## When NOT to use

- Input features already carry strand annotation — use Asymmetron's consecutive/contained/pairwise asymmetry functions directly instead
- Features do not overlap any strand-annotated element in the reference set, so orientation cannot be inferred from overlap
- The analysis goal is palindromic-sequence orientation or de novo motif discovery rather than orienting intervals against annotated elements

## Inputs

- Strand-unannotated genomic feature file (BED), e.g. gnomAD v2 germline structural variant breakpoints
- Strand-annotated reference elements (BED/table), e.g. LINE and SINE records from UCSC hg38 rmsk (11 March 2019 version)

## Outputs

- Strand-annotated feature file, e.g. oriented_breakpoints.bed with '+' or '-' assignment per feature
- Mapping of each feature to the closest overlapping strand-annotated element used for orientation

## How to apply

Use the Asymmetron orientation function (built on Pybedtools) to assign a '+' or '-' strand to each unannotated feature based on an overlapping strand-annotated element; when multiple elements overlap, keep the one with the minimal center-to-center distance |(s+e)/2 - (s'+e')/2|. In the germline SV use case, load gnomAD v2 breakpoints and the UCSC hg38 rmsk table (11 March 2019 version) filtered to LINE and SINE elements, then orient each breakpoint relative to the overlapping transposable element. The rationale is that orientation assignment is a prerequisite for computing background-corrected transcriptional (template/non-template via GENCODE v33 protein-coding genes) and replicative (leading/lagging via ENCODE BG02ES Repli-seq) asymmetry ratios. Write the strand-annotated features to a BED file (e.g. oriented_breakpoints.bed) so subsequent binomial tests can use each element class's background asymmetry probability.

## Related tools

- **Asymmetron** (Provides the orientation function that assigns '+'/'-' strand to unannotated features based on overlapping strand-assigned elements) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Implements the core BED-formatted file comparison underlying the overlap/closest-element selection)
- **bedtools** (Underlying 'bedtools closest' logic used by Asymmetron to determine the closest element when multiple candidates overlap)

## Examples

```
python orientation.py --unoriented gnomad_sv_breakpoints.bed --reference hg38_rmsk_LINE_SINE.bed --out oriented_breakpoints.bed
```

## Evaluation signals

- Every feature in oriented_breakpoints.bed carries a non-empty '+' or '-' strand field after orientation
- Each oriented feature overlaps a LINE/SINE element from the filtered rmsk annotation, and ties were resolved by minimal center-to-center distance |(s+e)/2 - (s'+e')/2|
- Downstream asymmetry computation succeeds: observed breakpoints are classifiable into template/non-template and leading/lagging categories
- Adjusted (background-corrected) strand asymmetry ratios per element class are finite and bounded in (0, 1)-type ratio ranges, with Bonferroni-corrected binomial P-values present for LINE and SINE
- Results reproduce the paper's direction of evidence: LINEs and SINEs show preference for the template strand (ratios 0.392 and 0.471, Bonferroni corrected P < 0.001)

## Limitations

- Orientation is only possible for features overlapping a strand-annotated element; unanchored features remain unassigned
- Python 3.5 environment with pybedtools/seaborn/numpy is specified in the README, so dependency/version mismatches may break the toolkit
- Transcriptional strand asymmetry of LINEs/SINEs varies with distance from the TSS, so background correction assumes element-class-level background probabilities rather than position-specific ones

## Evidence

- [methods] A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature: "A fourth function performs the strand assignment of an unassigned feature based on another overlapping feature"
- [readme] Asymmetron can assign strand orientation to third features: [evidence span withheld — anchor status: external_doc]
- [intro] germline SV orientation relative to transposable elements: "Using Asymmetron we show that germline structural variant breakpoints can be oriented relative to transposable elements"
- [methods] Pybedtools implementation: "its core BED-formatted file comparison functions use the package Pybedtools"
