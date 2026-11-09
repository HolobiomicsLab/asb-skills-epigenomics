---
name: transcription-factor-motif-occurrence-analysis
description: Use when the input is a genome-wide TF motif occurrence track (e.g. JASPAR
  PWM-derived BED with strand annotations) and the goal is to quantify orientation
  bias — same vs opposite strand — between consecutive or paired motif occurrences.
when_to_use_negative:
- The motif is palindromic — palindromic motifs cannot be oriented relative to one
  another, so same/opposite strand classification is meaningless (only non-palindromic
  motifs like CTCF are suitable).
- The input lacks strand annotations and no strand information can be assigned — Asymmetron
  requires DNA features provided with strand annotation (though the separate 'orientation'
  function can assign strand to a feature based on an overlapping strand-annotated
  feature).
- The question concerns strand asymmetry of features within regions or relative to
  transcription/replication rather than consecutive occurrences — use contained_asymmetries
  or pairwise_asymmetries instead.
edam_operation: http://edamontology.org/operation_3194
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3125
tools:
- name: Asymmetron
  role: Python toolkit providing the consecutive_patterns function for consecutive
    strand asymmetry estimation, permutation simulations for empirical P-values, and
    BED outputs of significant regions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Underpins Asymmetron's core BED-formatted file comparison functions (e.g.
    finding consecutive/closest features)
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Calculates the two-tailed binomial test P-values for strand asymmetry ratios
- name: FIMO
  role: Motif scanning with JASPAR PWMs against the human genome using human nucleotide
    frequencies as the background model, requiring minimum P-value <10^-6
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:2d95b19a3c1f3594b4e7174ad87228a76700b19f8a37c50ae883b6a9f057c6fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ahituv-lab/Asymmetron
  tools:
  - Asymmetron
  - FIMO
  - Pybedtools
  - Python
  - scipy
  edam_operation: http://edamontology.org/operation_3194
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3125
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# transcription-factor-motif-occurrence-analysis

## Summary

Workflow for analyzing the strand orientation of genome-wide transcription factor binding site (TFBS) motif occurrences using strand-annotated BED files and Asymmetron's consecutive strand asymmetry estimation, as demonstrated with CTCF (JASPAR MA0139.1) homotypic motif pairs.

## When to use

Use when the input is a genome-wide TF motif occurrence track (e.g. JASPAR PWM-derived BED with strand annotations) and the goal is to quantify orientation bias — same vs opposite strand — between consecutive or paired motif occurrences.

## When NOT to use

- The motif is palindromic — palindromic motifs cannot be oriented relative to one another, so same/opposite strand classification is meaningless (only non-palindromic motifs like CTCF are suitable).
- The input lacks strand annotations and no strand information can be assigned — Asymmetron requires DNA features provided with strand annotation (though the separate 'orientation' function can assign strand to a feature based on an overlapping strand-annotated feature).
- The question concerns strand asymmetry of features within regions or relative to transcription/replication rather than consecutive occurrences — use contained_asymmetries or pairwise_asymmetries instead.

## Inputs

- Strand-annotated BED file of TF motif occurrences (chromosome, start, end, strand sign + for forward/An, - for reverse-complement/Bn), e.g. derived from a JASPAR MA0139.1 CTCF motif track on hg19/hg38
- Optional: JASPAR PWM (e.g. from JASPAR 2020 CORE vertebrates non-redundant) and motif scan results (e.g. FIMO with human genome background model, minimum P-value <10^-6)

## Outputs

- Table of strand asymmetries for the same/opposite strand consecutive pattern (e.g. ctcf_consecutive_asymmetries.tsv) with ratios r = m/(m+o) and empirical two-tailed P-values
- BED file(s) of statistically significant consecutive regions

## How to apply

Convert the PWM-derived motif track (e.g. JASPAR MA0139.1 for CTCF on hg19/hg38) into a strand-annotated BED file with chromosome, start, end, and strand sign (+/-). Run Asymmetron's consecutive_patterns function with --max_distance (e.g. 100 bp for homotypic nearby CTCF pairs) and --min_distance 0 (default) so only same-chromosome motif pairs within the distance window are treated as consecutive. Classify each consecutive pair as same strand (m pairs) or opposite strand (o pairs), compute the strand asymmetry ratio r = m/(m+o), and derive a two-tailed empirical P-value via N simulations (e.g. N=100) that randomly permute strand labels while keeping +/− counts fixed. P-values can also be obtained with the two-tailed binomial test via scipy, and Bonferroni correction applied for multiple tests. This works only for non-palindromic motifs such as CTCF, whose orientation is detectable.

## Related tools

- **Asymmetron** (Python toolkit providing the consecutive_patterns function for consecutive strand asymmetry estimation, permutation simulations for empirical P-values, and BED outputs of significant regions) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Underpins Asymmetron's core BED-formatted file comparison functions (e.g. finding consecutive/closest features)) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Calculates the two-tailed binomial test P-values for strand asymmetry ratios)
- **FIMO** (Motif scanning with JASPAR PWMs against the human genome using human nucleotide frequencies as the background model, requiring minimum P-value <10^-6)

## Examples

```
python consecutive_patterns.py --input ctcf_MA0139.1_hg19.bed --max_distance 100 --min_distance 0 --simulations 100 --output ctcf_consecutive_asymmetries.tsv
```

## Evaluation signals

- The output table must show a same/opposite strand asymmetry ratio above 0.5 for CTCF together with a significant empirical P-value (P < 0.001 for CTCF within 100 bp), reproducing the reported homotypic same-strand orientation bias.
- The empirical P-value distribution from N=100 strand-label permutation simulations (keeping +/− counts fixed) should place the observed m vs o counts in the tail of the null distribution.
- Only motif pairs on the same chromosome with inter-motif distance within [--min_distance, --max_distance] (e.g. 0–100 bp) should appear as consecutive pairs in the output.
- The BED input schema is validated: each motif occurrence has chromosome, start, end, and a +/- strand sign; counts of + and − annotations are preserved between input and permutation simulation.
- For multiple TFs tested, P-values are Bonferroni corrected across tests.

## Limitations

- Detection of orientation bias requires a non-palindromic motif; palindromic motifs cannot be oriented.
- The default number of simulations is N=1000 (the CTCF task card used N=100), so empirical P-value resolution depends on the number of simulations chosen.
- The mechanism underlying TFBS orientation bias (e.g. preferential affinity for forward vs reverse-complement orientation, interaction/interference with RNA polymerase, steric hindrance between closely spaced TFBSs) is not established.

## Evidence

- [abstract] Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features.: "Asymmetron takes as input DNA features provided with strand annotation and outputs strand asymmetries for consecutive occurrences of a single DNA feature or between pairs of features."
- [methods] consecutive strand asymmetry estimation in a single file with strand annotation: "(i) consecutive strand asymmetry estimation in a single file with strand annotation"
- [methods] We then run N simulations (default: N = 1000), randomly assigning a value ('+' or '-') to every: [evidence span withheld — anchor status: not_found]
- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "It is a Python-based toolkit and its core BED-formatted file comparison functions use the package Pybedtools"
- [readme] The orientation of the CTCF motif dictates chromatin looping and three dimensional genome topology.: "An exemplar case reflecting the contribution of strand asymmetries in biological processes is the transcription factor CCCTC-binding factor (CTCF)."
- [abstract] the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions: "the orientation of the binding sites of 45% of human transcription factors analyzed have a significant DNA strand bias in transcribed regions"
