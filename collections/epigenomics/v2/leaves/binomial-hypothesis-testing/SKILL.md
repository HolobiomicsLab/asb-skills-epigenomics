---
name: binomial-hypothesis-testing
description: Use when counting strand-annotated feature occurrences (same vs opposite
  orientation) yields success/failure counts and you need a P-value for deviation
  from p = 0.5, e.g., motif-in-region strand bias tests.
when_to_use_negative:
- The data are continuous measurements rather than discrete same/opposite strand counts,
  so a binomial success/failure model does not apply
- You need empirical P-values that account for genomic covariates (e.g., regional
  GC or mappability) rather than a fixed p = 0.5 null; consider Asymmetron's permutation
  simulations instead
- You are testing a single feature with no multiplicity, so Bonferroni correction
  would be unnecessarily conservative
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Asymmetron
  role: Toolkit whose functions (contained_asymmetries, consecutive_patterns, pairwise_asymmetries)
    generate the same/opposite strand counts fed into the binomial test; performs
    Bonferroni correction of the resulting P-values
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Python package used to calculate the two-tailed binomial test P-values
- name: Pybedtools
  role: Underlies Asymmetron's core BED-formatted interval overlap comparisons that
    produce the orientation counts
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: numpy
  role: Python package used for the statistical analysis alongside scipy and pandas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:2228192f3e8567648ccef916249da6f0a057ff99561dc05997482e5155af4723
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Pybedtools
  - Python
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# binomial-hypothesis-testing

## Summary

Two-tailed binomial test used within Asymmetron to assess whether observed same/opposite strand asymmetry ratios of genomic features (e.g., Alu repeats within protein-coding genes) deviate significantly from an unbiased expectation, with Bonferroni correction for multiple tests.

## When to use

Use when counting strand-annotated feature occurrences (same vs opposite orientation) yields success/failure counts and you need a P-value for deviation from p = 0.5, e.g., motif-in-region strand bias tests.

## When NOT to use

- The data are continuous measurements rather than discrete same/opposite strand counts, so a binomial success/failure model does not apply
- You need empirical P-values that account for genomic covariates (e.g., regional GC or mappability) rather than a fixed p = 0.5 null; consider Asymmetron's permutation simulations instead
- You are testing a single feature with no multiplicity, so Bonferroni correction would be unnecessarily conservative

## Inputs

- Counts of same-strand and opposite-strand orientation occurrences from Asymmetron's contained_asymmetries / consecutive_patterns / pairwise_asymmetries functions
- Strand-annotated BED files of motifs (e.g., Alu repeats from UCSC hg38 rmsk.txt.gz) and regions (e.g., GENCODE v33 protein-coding genes)
- User-defined binomial probability (p = 0.5 for unbiased orientation)

## Outputs

- Two-tailed binomial test P-values per feature/region pair (scipy)
- Bonferroni-adjusted P-values for multiple tests
- Strand asymmetry ratios (same / (same + opposite))

## How to apply

After Asymmetron's contained_asymmetries (Pybedtools-based interval overlap) counts pairs in same versus opposite strand orientation between a motif file (e.g., UCSC hg38 RepeatMasker Alu SINE BED) and a region file (e.g., GENCODE v33 protein-coding gene BED), compute the asymmetry ratio as same-strand occurrences divided by same plus opposite occurrences. Evaluate the two-tailed binomial test with user-defined probability p = 0.5 using the scipy package in Python to obtain a P-value for deviation from unbiased orientation. When multiple features or regions are tested, adjust P-values with Bonferroni correction. Interpret ratios below 0.5 as preference for the template strand, as in the reported SINE/Alu ratio of 0.471 (Bonferroni-corrected P < 0.001). Asymmetron also supports permutation simulations (default N = 1000, randomly assigning '+' or '-' to every element) for empirical P-values when parametric assumptions are undesirable.

## Related tools

- **Asymmetron** (Toolkit whose functions (contained_asymmetries, consecutive_patterns, pairwise_asymmetries) generate the same/opposite strand counts fed into the binomial test; performs Bonferroni correction of the resulting P-values) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Python package used to calculate the two-tailed binomial test P-values)
- **Pybedtools** (Underlies Asymmetron's core BED-formatted interval overlap comparisons that produce the orientation counts) — https://github.com/Ahituv-lab/Asymmetron
- **numpy** (Python package used for the statistical analysis alongside scipy and pandas)

## Examples

```
from scipy.stats import binom_test; p = binom_test(same, same + opposite, 0.5)
```

## Evaluation signals

- Under a symmetric null (p = 0.5), the asymmetry ratio should be ~0.5; the reported SINE/Alu ratio of 0.471 indicates template strand preference and should replicate when reproducing the hg38 rmsk / GENCODE v33 analysis
- Bonferroni-corrected P-values for LINEs, SINEs and LTRs should be < 0.001 (reported ratios 0.392, 0.471, 0.316 respectively)
- Ratio must be bounded in [0, 1] and equal same / (same + opposite); verify counts sum to total overlapping occurrences from the Pybedtools overlap
- Strong transcriptional strand asymmetry should appear only in Alu repeats, not MIRs, when SINEs are subdivided, matching the article's finding
- Permutation-based empirical P-values (N = 1000 simulations, random '+'/'-' assignment) should be concordant in direction with the binomial P-values

## Limitations

- Bonferroni correction is conservative for large numbers of tests and can mask weaker but real asymmetries
- The binomial test with fixed p = 0.5 assumes no underlying background strand preference; the article notes that strand asymmetries must follow correction for background strand preferences to indicate mutational strand imbalance
- Asymmetron's parametric binomial P-value may not capture genomic heterogeneity; permutation simulations (default N = 1000) are offered as an alternative
- Weak strand asymmetry biases (e.g., replicative ones) are less pronounced than transcriptional biases and may be harder to detect; the article proposes dynamic changes in replication directionality during evolution as a possible explanation

## Evidence

- [methods] The corresponding P-values are calculated using the ‘scipy’ package in Python: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [methods] and are adjusted with Bonferroni correction in case of multiple tests.: "and are adjusted with Bonferroni correction in case of multiple tests."
- [methods] We then run N simulations (default: N = 1000), randomly assigning a value (‘+’ or ‘-‘) to every: [evidence span withheld — anchor status: not_found]
- [results] We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001): "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [readme] If the reference nucleotide or motif at the site of the mutation is found more frequently in one strand relative to the other, following correction for background strand preferences, it indicates a mutational strand asymmetry: "If the reference nucleotide or motif at the site of the mutation is found more frequently in one strand relative to the other, following correction for background strand preferences, it indicates a"
