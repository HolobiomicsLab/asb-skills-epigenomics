---
name: tfbs-coordinate-extraction-from-ucsc-track
description: Use when strand-annotated BED coordinates for named transcription factor
  motifs (e.g. SP1, YY1) must be pulled from a UCSC conserved TFBS track (tfbsConsSites)
  to feed strand asymmetry or motif-pairing analyses.
when_to_use_negative:
- The input is already a strand-annotated BED file of TF binding sites — no extraction
  from the UCSC track is needed.
- The analysis targets strand asymmetries of mutations or repeats relative to transcription/replication
  rather than pairs of TF motifs — use Asymmetron's consecutive_patterns or contained_asymmetries
  functions with the appropriate feature files instead.
- The motifs of interest are palindromic and thus cannot be oriented, since Asymmetron's
  orientation logic assumes non-palindromic sequences can be oriented relative to
  one another.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0749
tools:
- name: Asymmetron
  role: Downstream consumer of the extracted strand-annotated BED files; its pairwise_asymmetries
    function performs the strand asymmetry analysis these files enable
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Python wrapper through which Asymmetron's core BED-formatted file comparison
    functions are implemented on the extracted BED files
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: bedtools
  role: Provides the 'bedtools closest' function used to pair each element of the
    first BED file with its closest element in the second within the specified distance
    range
- name: Python
  role: Language of the Asymmetron toolkit and environment in which the extracted
    BED files are processed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:722fdc25c9c1b6acbd4a797e340404c02db6684fe8fc8adeff5e713f4826fe53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - bedtools
  - scipy
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tfbs-coordinate-extraction-from-ucsc-track

## Summary

Extracts chromosome, start, end and strand coordinates for specific transcription factor conserved binding sites (e.g. SP1, YY1) from the UCSC hg19 tfbsConsSites table and writes them as strand-annotated BED files, the required input format for Asymmetron's strand asymmetry functions.

## When to use

Use when strand-annotated BED coordinates for named transcription factor motifs (e.g. SP1, YY1) must be pulled from a UCSC conserved TFBS track (tfbsConsSites) to feed strand asymmetry or motif-pairing analyses.

## When NOT to use

- The input is already a strand-annotated BED file of TF binding sites — no extraction from the UCSC track is needed.
- The analysis targets strand asymmetries of mutations or repeats relative to transcription/replication rather than pairs of TF motifs — use Asymmetron's consecutive_patterns or contained_asymmetries functions with the appropriate feature files instead.
- The motifs of interest are palindromic and thus cannot be oriented, since Asymmetron's orientation logic assumes non-palindromic sequences can be oriented relative to one another.

## Inputs

- UCSC hg19 tfbsConsSites table (conserved transcription factor binding sites)
- Transcription factor name(s) to filter on (e.g. SP1, YY1)

## Outputs

- Strand-annotated BED file per transcription factor with chromosome, start, end, name, and strand columns (e.g. SP1_tfbsConsSites.bed, YY1_tfbsConsSites.bed)

## How to apply

Query the UCSC hg19 tfbsConsSites table and retain the coordinate fields needed for orientation analysis: chromosome, start, end and strand. Filter the table by the transcription factor name (e.g. SP1 or YY1) so that each factor yields its own file, then write each factor's sites as a strand-annotated BED file with the strand sign ('+'/'-') in the sixth column so bedtools can consume it. Build one BED per TF (e.g. SP1_tfbsConsSites.bed and YY1_tfbsConsSites.bed); the rationale is that Asymmetron's pairwise_asymmetries function requires two strand-assigned BED inputs and internally uses 'bedtools closest' to pair each element of the first file with its closest element in the second within a user-specified distance range (e.g. --min_distance=1 --max_distance=200). Correct application is confirmed when both BED files load in pybedtools, the downstream Asymmetron run completes, and pairwise_asymmetries_results.tsv is produced with same/opposite and convergent/divergent asymmetry ratios and binomial p-values.

## Related tools

- **Asymmetron** (Downstream consumer of the extracted strand-annotated BED files; its pairwise_asymmetries function performs the strand asymmetry analysis these files enable) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Python wrapper through which Asymmetron's core BED-formatted file comparison functions are implemented on the extracted BED files) — https://github.com/Ahituv-lab/Asymmetron
- **bedtools** (Provides the 'bedtools closest' function used to pair each element of the first BED file with its closest element in the second within the specified distance range)
- **Python** (Language of the Asymmetron toolkit and environment in which the extracted BED files are processed)

## Evaluation signals

- Each output BED file contains only the requested transcription factor's sites and includes a valid strand column restricted to '+' and '-' values.
- All coordinates fall within hg19 chromosome boundaries and respect start < end for every row.
- The BED files load without error via pybedtools and 'bedtools closest' accepts them as strand-aware input.
- The downstream run 'python pairwise_asymmetries.py SP1_tfbsConsSites.bed YY1_tfbsConsSites.bed --min_distance=1 --max_distance=200' completes and emits pairwise_asymmetries_results.tsv.
- The results table contains same/opposite and convergent/divergent asymmetry ratios with two-tailed binomial p-values (expected probability 0.5) and Bonferroni-corrected values.

## Limitations

- The tfbsConsSites track used in the tutorial is hg19, whereas the Asymmetron paper's own analyses were built on hg38; genome builds must be kept consistent across the pipeline.
- Only the closest element in the second BED file within the specified distance range is paired by 'bedtools closest', so alternative proximal partners are ignored.
- Orientation-based strand asymmetry is only meaningful for non-palindromic motifs, since palindromic sequences cannot be oriented relative to one another.

## Evidence

- [other] SP1 and YY1 conserved TFBSs are extracted from the UCSC hg19 tfbsConsSites table and analyzed with Asymmetron: [evidence span withheld — anchor status: not_found]
- [methods] bedtools closest pairing in Asymmetron: [evidence span withheld — anchor status: not_found]
- [abstract] Asymmetron input requirements: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [readme] palindromic sequence orientation caveat: "Sequences that are non palindromic can be oriented relative to one another."
- [readme] tutorial documentation pointer: "An in depth documentation, including a tutorial with examples can be found at:
https://asymmetron.readthedocs.io/en/latest/index.html"
