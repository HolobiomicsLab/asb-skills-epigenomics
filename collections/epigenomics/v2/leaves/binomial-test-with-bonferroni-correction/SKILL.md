---
name: binomial-test-with-bonferroni-correction
description: Use when a strand-orientation count (e.g. template vs non-template or
  vs lagging occurrences of breakpoints or motifs) must be tested against a background
  orientation probability, with multiple element classes tested simultaneously.
when_to_use_negative:
- Data are already strand-annotated and only orientation assignment is needed (use
  the Asymmetron orientation/Pybedtools step instead of this statistical test).
- The null hypothesis is a symmetric 0.5 probability but the feature class has a known
  intrinsic orientation preference — using the default binomial P without substituting
  the background probability would inflate significance.
- Permutation-based empirical P-values are required rather than a parametric binomial
  test (Asymmetron separately offers N=1000 permutation simulations).
edam_operation: http://edamontology.org/operation_2239
tools:
- name: Asymmetron
  role: Toolkit whose two-tailed binomial test and Bonferroni correction implement
    this skill; computes strand asymmetries of oriented features
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Python package used to calculate the two-tailed binomial P-values
- name: pandas
  role: Used with numpy for the surrounding statistical analysis and assembly of the
    output TSV
- name: numpy
  role: Used with pandas/scipy for the statistical analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:d631178714829ec2af0df0f71712ce77bb244125abd6eb1e0562b4fc5795b31f
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
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# binomial-test-with-bonferroni-correction

## Summary

Two-tailed binomial significance testing of strand asymmetry ratios, using each element class's background asymmetry probability as the null probability instead of the default 0.5, followed by Bonferroni correction for multiple tests. This is the statistical core used by Asymmetron to call transcriptional and replicative strand biases (e.g. of germline SV breakpoints at LINE/SINE elements).

## When to use

Use when a strand-orientation count (e.g. template vs non-template or leading vs lagging occurrences of breakpoints or motifs) must be tested against a background orientation probability, with multiple element classes tested simultaneously.

## When NOT to use

- Data are already strand-annotated and only orientation assignment is needed (use the Asymmetron orientation/Pybedtools step instead of this statistical test).
- The null hypothesis is a symmetric 0.5 probability but the feature class has a known intrinsic orientation preference — using the default binomial P without substituting the background probability would inflate significance.
- Permutation-based empirical P-values are required rather than a parametric binomial test (Asymmetron separately offers N=1000 permutation simulations).

## Inputs

- Observed counts per element class (e.g. breakpoints assigned '+' or '-' strand by the Asymmetron orientation function, as written to oriented_breakpoints.bed)
- Background asymmetry probability per element class (occurrences in non-template / leading strand orientation over total occurrences, from GENCODE v33 protein-coding and ENCODE BG02ES Repli-seq annotations)
- Number of element classes tested (for the Bonferroni divisor)

## Outputs

- Adjusted (observed/background) transcriptional and replicative strand asymmetry ratios per element class
- Bonferroni-corrected two-tailed binomial P-values per element class (columns of germline_sv_strand_asymmetries.tsv)

## How to apply

After computing observed and background strand asymmetry ratios (non-template / leading orientation occurrences over totals), derive the adjusted ratio by dividing the observed ratio by the background ratio for each element class (e.g. LINEs, SINEs). Compute two-tailed binomial P-values with the 'scipy' package in Python, replacing the default binomial probability P with each element class's background asymmetry probability so the null reflects the elements' intrinsic orientation preferences rather than a fair coin. Adjust all P-values with Bonferroni correction for the number of element classes tested. In the Asymmetron germline SV analysis this procedure yielded template-strand preferences with ratios 0.392 (LINEs), 0.471 (SINEs) and 0.316 (LTRs), all Bonferroni-corrected P < 0.001.

## Related tools

- **Asymmetron** (Toolkit whose two-tailed binomial test and Bonferroni correction implement this skill; computes strand asymmetries of oriented features) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Python package used to calculate the two-tailed binomial P-values)
- **pandas** (Used with numpy for the surrounding statistical analysis and assembly of the output TSV)
- **numpy** (Used with pandas/scipy for the statistical analysis)

## Examples

```
from scipy.stats import binom_test; p_adj = binom_test(k_nontemplate, n_total, p_background) * n_classes
```

## Evaluation signals

- Output table germline_sv_strand_asymmetries.tsv contains both adjusted transcriptional and replicative asymmetry ratios AND Bonferroni-corrected P-values for each element class (LINE, SINE).
- Each adjusted ratio equals the observed ratio divided by the corresponding element class background ratio (verifiable by recomputing observed and background ratios independently).
- Corrected P-values are the raw two-tailed binomial P-values (with P replaced by the background probability) multiplied by the number of classes tested, and fall within [0, 1].
- Replicating the paper's setting should recover template-strand preferences with transcriptional asymmetry ratios 0.392 (LINEs) and 0.471 (SINEs) at Bonferroni-corrected P < 0.001.
- Every tested class has a finite P-value; classes with zero observed counts should not be silently dropped.

## Limitations

- Bonferroni correction is conservative; only the number of tested element classes is accounted for.
- The test's validity depends on substituting the class-specific background asymmetry probability — using the default binomial probability when elements have intrinsic orientation preferences would misstate significance.
- Strand asymmetry magnitudes vary with genomic context (e.g. template-strand bias for LINEs/SINEs is more pronounced near the TSS and decreases with distance), so genome-wide aggregate ratios can mask regional effects.
- Weaker replicative than transcriptional asymmetry biases were observed, and dynamic changes in replication directionality during evolution are proposed as an explanation; replication-direction inference is cell-line specific (BG02ES).

## Evidence

- [methods] two-tailed binomial test: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] Bonferroni correction: "and are adjusted with Bonferroni correction in case of multiple tests."
- [methods] scipy implementation: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [results] template-strand preferences with reported ratios and corrected P-values: "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [intro] SV breakpoints oriented relative to transposable elements show asymmetries: "Using Asymmetron we show that germline structural variant breakpoints can be oriented relative to transposable elements and find transcriptional and replicative strand asymmetries in them"
