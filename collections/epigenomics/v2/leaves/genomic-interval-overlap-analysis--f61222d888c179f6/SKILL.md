---
name: genomic-interval-overlap-analysis
description: Use when you have strand-annotated BED files of motifs (e.g. PolyASite
  signals, RepeatMasker Alu elements, TFBSs) and regions (e.g. GENCODE protein-coding
  genes) and want to test whether motif orientation is biased relative to regions.
when_to_use_negative:
- Your features lack strand annotation or cannot be oriented — the analysis requires
  strand-assigned motifs and regions
- You need asymmetry between spatially adjacent (non-overlapping) elements rather
  than motifs contained within regions — use pairwise_asymmetries or consecutive_patterns
  instead
- You only have a single feature table without coordinate-based BED input, so no interval
  overlap can be computed
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function that performs the strand-aware
    overlap counting, binomial testing, and Bonferroni correction
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underlies the core BED-formatted file comparison (interval overlap) operations
    in contained_asymmetries
- name: scipy
  role: Computes the two-tailed binomial test P-values for same vs opposite orientation
    counts
- name: pandas
  role: Used to prepare/filter strand-annotated BED files (e.g. filtering GENCODE
    v33 to protein-coding genes)
- name: bedtools
  role: bedtools closest is used in related Asymmetron functionality to determine
    the closest element between BED files
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:db5c28044d3a851c819dbfb9c6864222d23ae5ffb6d3d32b6a2ede98c38051e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - Bedtools
  - Bowtie
  - Pybedtools
  - Python
  - bbduk
  - bedtools
  - genomation
  - matplotlib
  - numpy
  - pandas
  - samtools
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# genomic-interval-overlap-analysis

## Summary

A strand-aware interval overlap analysis, implemented in Asymmetron's contained_asymmetries function via Pybedtools, that classifies each motif-region overlap as same or opposite orientation, computes same/opposite strand asymmetry ratios, and evaluates significance with a two-tailed binomial test (expected p = 0.5) plus Bonferroni correction.

## When to use

Use when you have strand-annotated BED files of motifs (e.g. PolyASite signals, RepeatMasker Alu elements, TFBSs) and regions (e.g. GENCODE protein-coding genes) and want to test whether motif orientation is biased relative to regions.

## When NOT to use

- Your features lack strand annotation or cannot be oriented — the analysis requires strand-assigned motifs and regions
- You need asymmetry between spatially adjacent (non-overlapping) elements rather than motifs contained within regions — use pairwise_asymmetries or consecutive_patterns instead
- You only have a single feature table without coordinate-based BED input, so no interval overlap can be computed

## Inputs

- Strand-annotated regions BED file (e.g. GENCODE v33 protein-coding gene regions, hg38)
- Strand-annotated motifs BED file (e.g. PolyASite polyadenylation signal clusters, UCSC hg38 RepeatMasker rmsk.txt.gz Alu coordinates from 11 March 2019, Unibind/JASPAR 2020 TFBSs)
- expected_asym parameter (default 0.5)

## Outputs

- Counts of overlapping motif-region pairs in same vs opposite strand orientation
- Strand asymmetry ratio (same-strand occurrences / total overlapping occurrences)
- Two-tailed binomial test P-value per element (scipy)
- Bonferroni-corrected P-values for multiple tests

## How to apply

Format both inputs as strand-annotated BED files on the same genome build (hg38), filtering regions to protein-coding genes from GENCODE v33 and motifs to the element of interest. Run Asymmetron's contained_asymmetries function, which uses Pybedtools interval overlap comparisons to pair each overlapping motif with its containing region and classify the pair as same orientation when BED strand signs match, opposite otherwise. Compute the strand asymmetry ratio as same-strand occurrences divided by total overlapping occurrences, using the default expected same-strand probability of 0.5. Evaluate a two-tailed binomial test against p = 0.5 with scipy, then apply Bonferroni correction when multiple elements (e.g. 551 TF PWMs or LINE/SINE/LTR classes) are tested.

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function that performs the strand-aware overlap counting, binomial testing, and Bonferroni correction) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underlies the core BED-formatted file comparison (interval overlap) operations in contained_asymmetries)
- **scipy** (Computes the two-tailed binomial test P-values for same vs opposite orientation counts)
- **pandas** (Used to prepare/filter strand-annotated BED files (e.g. filtering GENCODE v33 to protein-coding genes))
- **bedtools** (bedtools closest is used in related Asymmetron functionality to determine the closest element between BED files)

## Examples

```
from asymmetron import contained_asymmetries; contained_asymmetries(regions='gencode_v33_protein_coding.bed', motifs='alu_hg38.bed', expected_asym=0.5)
```

## Evaluation signals

- PolyASite polyadenylation signals within GENCODE v33 protein-coding gene regions should reproduce a non-template strand asymmetry ratio of 0.72 with P = 2e-323
- SINE/Alu repeats from UCSC hg38 rmsk (11 March 2019) should show a template-strand preference (ratio 0.471, < 0.5) with Bonferroni-corrected P < 0.001
- For 551 TF PWMs in transcribed regions, ~45% (248) should show significant Bonferroni-corrected strand bias (binomial P < 0.05, corrected)
- The asymmetry ratio must lie in [0, 1] with ratio > 0.5 indicating non-template/same-strand enrichment and < 0.5 template-strand enrichment under expected_asym = 0.5
- Every counted pair must derive from an actual interval overlap between a motif and a region with matching chromosomes and coordinates (Pybedtools overlap check)

## Limitations

- Binomial testing assumes an expected same-strand probability of 0.5; deviations from strand-symmetric background may bias results
- Bonferroni correction across many elements (e.g. 551 PWMs) is conservative and may miss weaker asymmetries
- The conda environment pins python=3.5 with pybedtools/seaborn/numpy, so newer Python versions may require environment adaptation
- README notes the toolkit requires palindromic-aware interpretation: only non-palindromic sequences can be oriented relative to one another

## Evidence

- [methods] Pybedtools dependency for BED comparison: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] Contained asymmetries function scope: "(ii) strand asymmetry estimation of strand-assigned motifs within strand-assigned regions"
- [methods] Binomial P-value computation: "The corresponding P-values are calculated using the ‘scipy’ package in Python"
- [methods] Multiple testing correction: "and are adjusted with Bonferroni correction in case of multiple tests."
- [readme] README summary of overlap capabilities: [evidence span withheld — anchor status: external_doc]
- [results] Reproduced SINE result as validation anchor: "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
