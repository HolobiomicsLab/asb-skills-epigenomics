---
name: motif-orientation-pairwise-classification
description: Use when you have a strand-annotated BED file of motif occurrences (e.g.
  JASPAR MA0139.1 CTCF sites in hg19) and need to test whether nearby consecutive
  pairs within a distance window show same-strand versus opposite-strand orientation
  bias.
when_to_use_negative:
- The motif or feature is palindromic — sequences that are non palindromic can be
  oriented relative to one another, so palindromic motifs carry no orientation information
  and same/opposite classification is meaningless
- Input features lack strand annotation — use Asymmetron's separate 'orientation'
  function (strand assignment of an unassigned feature based on another overlapping
  feature) instead of consecutive orientation classification
- The biological question concerns motifs within strand-assigned regions or between
  two different motifs — use Asymmetron's contained_asymmetries or pairwise_asymmetries
  functions rather than consecutive_patterns
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0654
tools:
- name: Asymmetron
  role: Python toolkit whose consecutive_patterns function classifies consecutive
    occurrences of a single strand-annotated feature into same/opposite strand pairs,
    computes the asymmetry ratio, and runs permutation simulations for empirical P-values
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underpins Asymmetron's core BED-formatted file comparison functions used to
    find consecutive same-chromosome motif pairs
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Python
  role: Implementation language of the toolkit and for downstream statistical analysis
    with math, scipy, pandas and numpy
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Used to calculate P-values for the two-tailed binomial test on the same/opposite
    pair counts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:0058c5975b00d35ff4d7775586206364b9ad6af3fc45a25e29531a7d1df5e089
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - scipy
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# motif-orientation-pairwise-classification

## Summary

A skill for classifying the relative strand orientation of consecutive occurrences of a single strand-annotated DNA motif (e.g. CTCF MA0139.1) into same-strand (m) versus opposite-strand (o) pairs, computing the strand asymmetry ratio r = m/(m+o), and testing its significance with Asymmetron's consecutive_patterns function.

## When to use

Use when you have a strand-annotated BED file of motif occurrences (e.g. JASPAR MA0139.1 CTCF sites in hg19) and need to test whether nearby consecutive pairs within a distance window show same-strand versus opposite-strand orientation bias.

## When NOT to use

- The motif or feature is palindromic — sequences that are non palindromic can be oriented relative to one another, so palindromic motifs carry no orientation information and same/opposite classification is meaningless
- Input features lack strand annotation — use Asymmetron's separate 'orientation' function (strand assignment of an unassigned feature based on another overlapping feature) instead of consecutive orientation classification
- The biological question concerns motifs within strand-assigned regions or between two different motifs — use Asymmetron's contained_asymmetries or pairwise_asymmetries functions rather than consecutive_patterns

## Inputs

- Strand-annotated BED file of motif occurrences (chromosome, start, end, strand sign; '+' for An, '-' for Bn), e.g. derived from a JASPAR MA0139.1 CTCF motif track for hg19
- Distance window parameters (--max_distance, --min_distance) specifying which same-chromosome motif pairs are consecutive
- Number of permutation simulations for empirical P-value estimation (default N=1000 in the paper)

## Outputs

- TSV table of strand asymmetries for the same/opposite strand pattern (e.g. ctcf_consecutive_asymmetries.tsv) containing m pairs, o pairs, asymmetry ratio r = m/(m+o), and empirical P-value
- BED file(s) of statistically significant consecutive regions

## How to apply

Convert the motif occurrence track (e.g. the genome-wide CTCF track for the JASPAR MA0139.1 position frequency matrix) into a strand-annotated BED file with chromosome, start, end, and strand sign (+/-). Run Asymmetron's consecutive_patterns function with --max_distance 100 and --min_distance 0 (defaults) so only motif pairs on the same chromosome within the distance window are treated as consecutive. Classify each consecutive pair as same strand (m pairs) or opposite strand (o pairs), compute the strand asymmetry ratio r = m/(m+o), and derive a two-tailed empirical P-value by randomly permuting strand labels while keeping the +/- counts fixed (N simulations; the paper's default is N=1000). Apply Bonferroni correction when multiple motifs or windows are tested. Validation requires a non-palindromic motif (so asymmetries are detectable), an asymmetry ratio above 0.5, and a significant empirical P-value (e.g. < 0.001 as reported for the CTCF same-orientation bias).

## Related tools

- **Asymmetron** (Python toolkit whose consecutive_patterns function classifies consecutive occurrences of a single strand-annotated feature into same/opposite strand pairs, computes the asymmetry ratio, and runs permutation simulations for empirical P-values) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underpins Asymmetron's core BED-formatted file comparison functions used to find consecutive same-chromosome motif pairs) — https://github.com/Ahituv-lab/Asymmetron
- **Python** (Implementation language of the toolkit and for downstream statistical analysis with math, scipy, pandas and numpy) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Used to calculate P-values for the two-tailed binomial test on the same/opposite pair counts)

## Examples

```
python consecutive_patterns.py --input ctcf_MA0139.1_strand.bed --max_distance 100 --min_distance 0 --simulations 100 --output ctcf_consecutive_asymmetries.tsv
```

## Evaluation signals

- The output TSV shows a same/opposite strand asymmetry ratio above 0.5 (e.g. CTCF consecutive occurrences within 100 bp prefer the same orientation) together with a significant empirical P-value (< 0.001)
- Only motif pairs on the same chromosome within the --min_distance to --max_distance window are treated as consecutive (e.g. 0-100 bp)
- Permutation simulations permute strand labels while keeping the + and - counts fixed; the empirical P-value is two-tailed and Bonferroni-adjusted when multiple tests are run
- The input BED file carries explicit strand signs for every occurrence and the motif is confirmed non-palindromic (as CTCF MA0139.1 is), otherwise asymmetry is undetectable
- Significant consecutive regions are emitted as BED files whose coordinates match the original motif occurrences

## Limitations

- Orientation classification is only meaningful for non-palindromic motifs; palindromic motifs yield no detectable asymmetry
- The empirical P-value depends on the number of simulations (paper default N = 1000; the CTCF example used N = 100), so low simulation counts limit P-value resolution
- Asymmetry interpretation is confounded by distance from reference features (e.g. transcriptional strand biases of LINEs/SINEs decrease with distance from the TSS), and P-values require Bonferroni correction across multiple tests
- The mechanism underlying observed orientation preferences (e.g. steric hindrance between closely spaced TFBSs or preferential affinity for forward versus reverse-complement orientations) remains unestablished

## Evidence

- [abstract] Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features.: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [readme] Sequences that are non palindromic can be oriented relative to one another.: "Sequences that are non palindromic can be oriented relative to one another."
- [readme] The orientation of the CTCF motif dictates chromatin looping and three dimensional genome topology.: "The orientation of the CTCF motif dictates chromatin looping and three dimensional genome topology."
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] and are adjusted with Bonferroni correction in case of multiple tests.: "and are adjusted with Bonferroni correction in case of multiple tests."
