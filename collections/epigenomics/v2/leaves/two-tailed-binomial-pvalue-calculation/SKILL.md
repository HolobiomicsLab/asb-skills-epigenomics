---
name: two-tailed-binomial-pvalue-calculation
description: Use when you have classified strand-oriented genomic events (e.g. same/opposite
  or convergent/divergent motif orientations) into two classes and need a significance
  test against a null expectation of 0.5, with multiple-testing correction.
when_to_use_negative:
- The comparison involves more than two categories (e.g. replication-timing bins or
  multi-class orientations), where a binomial test against p = 0.5 does not apply.
- Feature positions or strand assignments are so spatially/structurally biased that
  a plain binomial null of 0.5 is inappropriate — use the permutation simulation mode
  instead of the plain binomial P-value.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Toolkit whose consecutive_patterns, contained_asymmetries and pairwise_asymmetries
    functions produce the orientation counts fed to the binomial test
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Python package used to calculate the two-tailed binomial test P-values
- name: numpy
  role: Used alongside scipy/pandas for statistical analysis and permutation simulations
- name: pandas
  role: Used for statistical analysis and assembly of asymmetry results tables
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:c53c3fcfdc9d354cb08fae0bcf31b5d6340555ec37ba6252699dcac11d6ffefc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# two-tailed-binomial-pvalue-calculation

## Summary

Statistical testing procedure from Asymmetron that computes two-tailed binomial P-values (expected probability 0.5) on strand-orientation counts and adjusts them with Bonferroni correction for multiple tests, optionally complemented by permutation simulations for empirical P-values.

## When to use

Use when you have classified strand-oriented genomic events (e.g. same/opposite or convergent/divergent motif orientations) into two classes and need a significance test against a null expectation of 0.5, with multiple-testing correction.

## When NOT to use

- The comparison involves more than two categories (e.g. replication-timing bins or multi-class orientations), where a binomial test against p = 0.5 does not apply.
- Feature positions or strand assignments are so spatially/structurally biased that a plain binomial null of 0.5 is inappropriate — use the permutation simulation mode instead of the plain binomial P-value.

## Inputs

- Counts of occurrences in each of two orientation classes (e.g. same vs opposite strand; convergent vs divergent) from Asymmetron consecutive/contained/pairwise asymmetry analysis
- Optional: raw strand-annotated BED files of motifs and regions used to generate those counts

## Outputs

- Two-tailed binomial P-values per asymmetry ratio (scipy)
- Bonferroni-corrected P-values for multiple tests
- Optional empirical P-values from N = 1000 permutation simulations with randomized strand assignment
- Asymmetry ratios (occurrences-in-class / total occurrences) written to results tables such as pairwise_asymmetries_results.tsv

## How to apply

After counting occurrences in each of the two orientation classes (e.g. same-strand vs opposite-strand overlap of PolyASite polyadenylation signals in GENCODE v33 protein-coding gene regions, or same/opposite and convergent/divergent pairs of SP1 and YY1 TFBSs at 1-200 bp), calculate the P-value for the two-tailed binomial test using the 'scipy' package in Python with expected probability p = 0.5, since each feature is assumed equally likely to be in either orientation under the null. The asymmetry ratio is computed as occurrences-in-class divided by total occurrences. Adjust the resulting P-values with Bonferroni correction when multiple tests (e.g. multiple motifs or distance bins) are evaluated. For robustness against biased feature distributions, run N simulations (default N = 1000) that randomly assign a '+' or '-' strand value to every element to obtain empirical P-values. Statistical analysis is performed in Python with 'math', 'scipy', 'pandas' and 'numpy'.

## Related tools

- **Asymmetron** (Toolkit whose consecutive_patterns, contained_asymmetries and pairwise_asymmetries functions produce the orientation counts fed to the binomial test) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Python package used to calculate the two-tailed binomial test P-values)
- **numpy** (Used alongside scipy/pandas for statistical analysis and permutation simulations)
- **pandas** (Used for statistical analysis and assembly of asymmetry results tables)

## Examples

```
python pairwise_asymmetries.py SP1_tfbsConsSites.bed YY1_tfbsConsSites.bed --min_distance=1 --max_distance=200
```

## Evaluation signals

- Recomputed P-value matches a reported value for the same input, e.g. polyadenylation signal strand asymmetry ratio 0.72 with P-value = 2e-323 in GENCODE v33 protein-coding regions.
- Bonferroni-corrected P-values for LINE/SINE/LTR template-strand preference fall below 0.001 (ratios 0.392, 0.471, 0.316).
- Asymmetry ratio values are confined to [0, 1], computed as occurrences-in-class divided by total occurrences.
- Permutation-based empirical P-values (N = 1000 random '+'/'-' strand assignments) are consistent in direction of significance with the parametric binomial P-values.
- Output tables (e.g. pairwise_asymmetries_results.tsv) contain asymmetry ratios paired with binomial P-values for each tested comparison.

## Limitations

- The default binomial null assumes expected same-strand probability of 0.5; datasets with background strand-composition bias violate this assumption.
- Multiple simultaneous tests (e.g. 551 PWMs or many distance bins) reduce power after Bonferroni correction.
- Significance of an asymmetry does not establish mechanism — e.g. TFBS orientation preference may reflect preferential affinity for forward vs reverse-complement orientations or interference with RNA polymerase, which remains unestablished.

## Evidence

- [methods] The corresponding P-values are calculated using the ‘scipy’ package in Python: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [methods] We then calculate the P-value for the two-tailed binomial test as follows: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] Bonferroni correction for multiple tests: "and are adjusted with Bonferroni correction in case of multiple tests."
- [methods] We then run N simulations (default: N = 1000), randomly assigning a value (‘+’ or ‘-‘) to every: [evidence span withheld — anchor status: not_found]
- [methods] Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’: "Statistical analysis was performed in Python with the packages ‘math’, ‘scipy’, ‘pandas’ and ‘numpy’"
