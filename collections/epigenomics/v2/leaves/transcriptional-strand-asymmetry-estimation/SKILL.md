---
name: transcriptional-strand-asymmetry-estimation
description: Use when strand-annotated motifs (BED files with '+'/'-' strands) must
  be tested for same-vs-opposite orientation bias relative to template/non-template
  direction of strand-annotated regions such as GENCODE protein-coding genes.
when_to_use_negative:
- Motifs are palindromic (e.g. some restriction-site-like sequences) so a '+'/'-'
  strand orientation cannot be meaningfully assigned
- The question is replicative rather than transcriptional asymmetry, or the features
  are unassigned and must first be oriented relative to overlapping elements via Asymmetron's
  orientation function
- Genic region overlap counts alone cannot disentangle motif orientation from element-class
  background strand preference and no background correction data is available
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: Asymmetron
  role: Provides the contained_asymmetries function that counts same-vs-opposite strand
    orientation of motifs contained within strand-assigned regions
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: Pybedtools
  role: Backend for Asymmetron's BED-formatted interval overlap comparisons
  repo: https://github.com/Ahituv-lab/Asymmetron
- name: scipy
  role: Two-tailed binomial test P-value calculation against p=0.5 or background asymmetry
    probability
- name: pandas
  role: Filtering GENCODE v33 to protein-coding genes and formatting strand-annotated
    BED files; statistical analysis
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkaa1052
    title: 'Asymmetron: a toolkit for the identification of strand asymmetry patterns
      in biological sequences'
  source_content_hash: sha256:91e93199dfb6671f7ae0a817bda5cacb3f285b1aadaa0beb8e7a9ff64b097b32
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# transcriptional-strand-asymmetry-estimation

## Summary

Estimates transcriptional strand asymmetry of strand-annotated motifs (e.g. Alu/SINE repeats from UCSC hg38 rmsk, PolyASite PAS clusters) within strand-annotated genic regions (GENCODE v33 protein-coding genes) using Asymmetron's contained_asymmetries interval-overlap function, a two-tailed binomial test against p=0.5, and Bonferroni correction.

## When to use

Use when strand-annotated motifs (BED files with '+'/'-' strands) must be tested for same-vs-opposite orientation bias relative to template/non-template direction of strand-annotated regions such as GENCODE protein-coding genes.

## When NOT to use

- Motifs are palindromic (e.g. some restriction-site-like sequences) so a '+'/'-' strand orientation cannot be meaningfully assigned
- The question is replicative rather than transcriptional asymmetry, or the features are unassigned and must first be oriented relative to overlapping elements via Asymmetron's orientation function
- Genic region overlap counts alone cannot disentangle motif orientation from element-class background strand preference and no background correction data is available

## Inputs

- Strand-annotated regions BED file (GENCODE v33 filtered to protein-coding genes, hg38)
- Strand-annotated motifs BED file (chromosome, start, end, strand), e.g. Alu/SINE/LINE/LTR elements from UCSC hg38 rmsk (11 March 2019 version) or PolyASite PAS clusters
- Optional background asymmetry probability per element class for adjusted binomial testing

## Outputs

- Same/opposite strand orientation counts per region-motif pair class
- Transcriptional strand asymmetry ratio (same-strand occurrences / total overlapping occurrences)
- Two-tailed binomial test P-values (scipy, p=0.5 or class-specific background probability) with Bonferroni correction
- Summary table (e.g. TSV) of asymmetry ratios and corrected P-values per element class

## How to apply

Build a strand-annotated regions BED file by filtering GENCODE v33 to protein-coding genes and a strand-annotated motifs BED file (e.g. Alu/SINE coordinates from UCSC hg38 rmsk.txt.gz, 11 March 2019 version) using pandas. Run Asymmetron's contained_asymmetries function (Pybedtools-based interval overlap) to classify each motif-region overlap pair as same or opposite strand orientation, then compute the asymmetry ratio as occurrences in same orientation divided by total overlapping occurrences. Evaluate a two-tailed scipy binomial test with probability p=0.5 and apply Bonferroni correction for multiple tests. Note the ratio convention: template-strand preference yields ratio < 0.5, non-template preference ratio > 0.5. When motif orientation is confounded by background element-level strand preferences (as for SV breakpoints relative to LINEs/SINEs), first compute a background asymmetry probability per element class and use it in place of the default p=0.5 in the binomial test.

## Related tools

- **Asymmetron** (Provides the contained_asymmetries function that counts same-vs-opposite strand orientation of motifs contained within strand-assigned regions) — https://github.com/Ahituv-lab/Asymmetron
- **Pybedtools** (Backend for Asymmetron's BED-formatted interval overlap comparisons) — https://github.com/Ahituv-lab/Asymmetron
- **scipy** (Two-tailed binomial test P-value calculation against p=0.5 or background asymmetry probability)
- **pandas** (Filtering GENCODE v33 to protein-coding genes and formatting strand-annotated BED files; statistical analysis)

## Examples

```
python -c "from asymmetron.asymmetron import contained_asymmetries; contained_asymmetries(regions='gencode_v33_pc_genes.bed', motifs='alu_sine_hg38.bed', p=0.5)"
```

## Evaluation signals

- Ratio invariance check: ratio = same-strand occurrences / (same + opposite occurrences) over all overlapping pairs, so ratio must lie in [0,1]
- Reproduce the reported SINE transcriptional asymmetry ratio of 0.471 towards the template strand (Bonferroni-corrected P < 0.001); within SINEs, asymmetry should localize to Alu repeats and not MIRs
- Reproduce the PolyASite PAS asymmetry ratio of 0.72 towards the non-template strand with P = 2e-323 in protein-coding genic regions
- Bonferroni-corrected P-values must be computed from the same two-tailed binomial test used for the raw P-values, with the number of tests matching the number of element classes/regions tested
- Strand coverage check: every motif-region overlap pair is classified as same or opposite orientation — no pairs dropped due to missing strand annotations

## Limitations

- Default expected probability p=0.5 may be inappropriate when motifs themselves carry background strand preferences; task-derived guidance replaces p with the element class's background asymmetry probability before Bonferroni correction
- Asymmetry strength depends on distance from the TSS: LINEs and SINEs show a more pronounced template-strand bias near the TSS that decreases with distance, and LTRs showed no such correlation, so region definitions affect sensitivity
- Requires strand annotation on both regions and motifs; unannotated features must first be oriented via Asymmetron's separate orientation function

## Evidence

- [methods] its core BED-formatted file comparison functions use the package Pybedtools: "its core BED-formatted file comparison functions use the package Pybedtools"
- [methods] We then calculate the P-value for the two-tailed binomial test as follows: "We then calculate the P-value for the two-tailed binomial test as follows"
- [methods] and are adjusted with Bonferroni correction in case of multiple tests.: "and are adjusted with Bonferroni correction in case of multiple tests."
- [results] We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001): "We found a preference for LINEs, SINEs and LTRs to be at the template strand (ratios 0.392, 0.471, 0.316, respectively, binomial test, Bonferroni corrected P-values < 0.001)"
- [methods] Gene annotation from GENCODE was used (v33); the file was derived from (https://www.gencodegenes.org/): "Gene annotation from GENCODE was used (v33); the file was derived from (https://www.gencodegenes.org/)"
- [readme] Asymmetron can assign strand orientation to third features such as mutations, by orienting them relative to other genomic elements.: [evidence span withheld — anchor status: external_doc]
