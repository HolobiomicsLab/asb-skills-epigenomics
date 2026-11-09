---
name: strand-asymmetry-estimation-of-consecutive-motif-occurrences
description: Use when you have a BED file of strand-annotated occurrences of one non-palindromic
  genomic feature and want to test whether nearby consecutive pairs show an orientation
  bias (same vs opposite strand) within a defined distance range.
when_to_use_negative:
- The feature is palindromic or its orientation cannot be assigned — non-palindromic
  sequences are required to be oriented relative to one another (the paper cites CTCF
  as the exemplar non-palindromic case).
- The question concerns asymmetry of motifs within strand-assigned regions (transcription/replication
  orientation) rather than between consecutive occurrences of the same feature — use
  contained_asymmetries or pairwise_asymmetries instead.
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0654
tools:
- name: Asymmetron
  role: Python toolkit providing the consecutive_patterns function for consecutive
    strand asymmetry estimation and empirical permutation P-values
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted file comparison functions (e.g.,
    finding consecutive pairs on the same chromosome)
- name: scipy
  role: Used to calculate the two-tailed binomial test P-values
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:c4526b0e788eab283ea473977134fcefd54c112795c62ae98b975ef1389da702
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - scipy
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# strand-asymmetry-estimation-of-consecutive-motif-occurrences

## Summary

Quantifies whether consecutive occurrences of a single strand-annotated genomic element (e.g., CTCF motif instances) are preferentially in the same or opposite orientation within a distance window, using Asymmetron's consecutive_patterns function with empirical permutation P-values.

## When to use

Use when you have a BED file of strand-annotated occurrences of one non-palindromic genomic feature and want to test whether nearby consecutive pairs show an orientation bias (same vs opposite strand) within a defined distance range.

## When NOT to use

- The feature is palindromic or its orientation cannot be assigned — non-palindromic sequences are required to be oriented relative to one another (the paper cites CTCF as the exemplar non-palindromic case).
- The question concerns asymmetry of motifs within strand-assigned regions (transcription/replication orientation) rather than between consecutive occurrences of the same feature — use contained_asymmetries or pairwise_asymmetries instead.

## Inputs

- Strand-annotated BED file of motif/genomic feature occurrences (chromosome, start, end, strand '+' or '-')
- Motif occurrence track for a specific position frequency matrix (e.g., JASPAR MA0139.1 CTCF) and genome build (e.g., hg19/hg38)
- Distance window parameters (--max_distance, e.g., 100 bp; --min_distance, default 0)
- Number of permutation simulations (default N = 1000)

## Outputs

- TSV table of strand asymmetries for the same/opposite strand consecutive pattern (e.g., ctcf_consecutive_asymmetries.tsv) including ratio r = m/(m+o) and empirical P-value
- BED files of statistically significant consecutive regions

## How to apply

Convert the motif track (e.g., JASPAR MA0139.1 CTCF occurrences for hg19) into a strand-annotated BED file with chromosome, start, end, and strand ('+' or '-'). Run Asymmetron's consecutive_patterns with --max_distance (e.g., 100 bp) and --min_distance 0 (default) so only motif pairs on the same chromosome within the window are treated as consecutive. Classify each consecutive pair as same strand (m pairs) or opposite strand (o pairs), compute the strand asymmetry ratio r = m/(m+o), and derive an empirical two-tailed P-value from N simulations (the paper's default is N=1000; the task card used N=100) that randomly permute strand labels while keeping +/− counts fixed. P-values come from the scipy package and are adjusted with Bonferroni correction for multiple tests. Outputs include a table of same/opposite strand asymmetries (e.g., ctcf_consecutive_asymmetries.tsv) plus BED files of statistically significant consecutive regions; an asymmetry ratio above 0.5 with a significant empirical P-value indicates a same-strand orientation bias.

## Related tools

- **Asymmetron** (Python toolkit providing the consecutive_patterns function for consecutive strand asymmetry estimation and empirical permutation P-values) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies Asymmetron's core BED-formatted file comparison functions (e.g., finding consecutive pairs on the same chromosome))
- **scipy** (Used to calculate the two-tailed binomial test P-values)

## Examples

```
python consecutive_patterns.py --input ctcf_MA0139.1_hg19.bed --max_distance 100 --min_distance 0 --simulations 100 --output ctcf_consecutive_asymmetries.tsv
```

## Evaluation signals

- The same/opposite strand asymmetry ratio must be above 0.5 for CTCF (a non-palindromic motif), reproducing the reported homotypic same-strand orientation bias.
- The empirical two-tailed P-value must reach significance (task card reports P < 0.001 with N=100 strand-label permutation simulations).
- Only motif pairs on the same chromosome within the --min_distance/--max_distance window are counted as consecutive (sanity-check pair counts against distance filtering).
- Permutation simulations must randomly permute strand labels while keeping the total '+' and '−' counts fixed, so the empirical null preserves strand composition.
- Output table schema must contain same-strand (m), opposite-strand (o) counts, the asymmetry ratio, and Bonferroni-adjusted P-values where multiple tests are run.

## Limitations

- Detectability depends on the motif being non-palindromic; palindromic motifs yield no strand asymmetry signal.
- Default is N = 1000 simulations; using fewer (e.g., N=100) limits empirical P-value resolution.
- The article notes it remains unknown how closely spaced TFBS orientation influences steric hindrance and competition for binding, so a significant bias does not by itself establish mechanism.

## Evidence

- [abstract] Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features.: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [methods] We then run N simulations (default: N = 1000), randomly assigning a value (‘+’ or ‘-‘) to every: [evidence span withheld — anchor status: not_found]
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [readme] Sequences that are non palindromic can be oriented relative to one another. An exemplar case reflecting the contribution of strand asymmetries in biological processes is the transcription factor CCCTC-binding factor (CTCF).: "Sequences that are non palindromic can be oriented relative to one another. An exemplar case reflecting the contribution of strand asymmetries in biological processes is the transcription factor"
- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions"
