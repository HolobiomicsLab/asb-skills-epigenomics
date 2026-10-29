---
name: bias-correction-evaluation
description: Use when a RNA-Seq expression-estimation method has been run with and
  without fragment-bias correction and the improvement must be validated against an
  independent qRT-PCR truth set (e.g., 907 uniquely probed MAQC HBR transcripts).
when_to_use_negative:
- When no independently measured qRT-PCR (or equivalent external truth set) is available
  — the R2-vs-qRT-PCR comparison cannot be computed
- When the goal is to estimate bias parameters themselves rather than evaluate whether
  an existing bias-corrected estimator improves accuracy
- When transcripts cannot be restricted to a uniquely probed subset, since multi-mapping
  probes inflate or deflate R2 spuriously
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3512
tools:
- name: Cufflinks
  role: Estimate transcript expression from mapped MAQC HBR alignments both with and
    without fragment-bias correction enabled
- name: Genominator
  role: Count-based baseline method run with bias parameters learned per chromosome
    to produce uncorrected/corrected comparison R2 values
- name: mseq
  role: MART-regression-based baseline whose model is trained on the 100 most highly
    expressed transcripts after Bowtie remapping
- name: Bowtie
  role: Remap MAQC HBR reads to the transcriptome before mseq-based expression estimation
- name: Python
  role: Converts standard SAM and FASTA input into the mseq input format and computes
    R2/permutation-test statistics
- name: NanoString
  role: Supplementary expression measurements included alongside the qRT-PCR comparison
    data
- name: eXpress-D
  role: Distributed ambiguous fragment assignment using eXpress and Spark, for likelihood-based
    simultaneous bias and expression estimation at scale
  repo: github.com/adarob/eXpress-d
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:a9e0346fbbec037e44408c0f58744af586abdc24689906ca6e0f962c45320552
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bowtie
  - Cufflinks
  - Genominator
  - NanoString
  - Python
  - eXpress-D
  - mseq
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bias-correction-evaluation

## Summary

A benchmark-driven skill for quantifying whether RNA-Seq fragment-bias correction improves expression estimates, by computing squared Pearson correlations (R2) between corrected and uncorrected transcript abundance estimates and independently measured qRT-PCR reference values on the MAQC HBR dataset.

## When to use

Use when a RNA-Seq expression-estimation method has been run with and without fragment-bias correction and the improvement must be validated against an independent qRT-PCR truth set (e.g., 907 uniquely probed MAQC HBR transcripts).

## When NOT to use

- When no independently measured qRT-PCR (or equivalent external truth set) is available — the R2-vs-qRT-PCR comparison cannot be computed
- When the goal is to estimate bias parameters themselves rather than evaluate whether an existing bias-corrected estimator improves accuracy
- When transcripts cannot be restricted to a uniquely probed subset, since multi-mapping probes inflate or deflate R2 spuriously

## Inputs

- Mapped RNA-Seq alignments (e.g., SAM/BAM from SRA012427 MAQC HBR reads, remapped with Bowtie)
- Independently measured qRT-PCR (TaqMan) reference expression values for MAQC HBR transcripts
- Transcript FASTA sequences and SAM alignments convertible to mseq input format via the supplementary Python script
- Uniqueness-of-probe mapping information for filtering to 907 uniquely probed transcripts

## Outputs

- Uncorrected and corrected R2 values per method (e.g., Cufflinks 0.753 -> 0.807; mseq 0.730 -> 0.755; Genominator 0.711 -> 0.715)
- Permutation-test P-value for the corrected-vs-uncorrected correlation improvement (e.g., P = 0.0007)
- CSV result files such as qpcr_correlation_results.csv and comparison_r2_values.csv

## How to apply

Map sample alignments (e.g., SRA012427 MAQC Human Brain Reference reads) to the transcriptome with a aligner such as Bowtie, then run the estimator of interest twice: once without bias correction and once with fragment-bias correction enabled (e.g., Cufflinks -b). Load the independently performed qRT-PCR measurements and filter to the transcripts whose TaqMan probes map uniquely (907 in the MAQC HBR benchmark). Compute R2 between qRT-PCR and both uncorrected and corrected estimates over the identical transcript set, and assess significance of the improvement with a permutation test comparing the two correlations. As cross-method baselines, run Genominator with bias parameters learned per chromosome (expected R2 0.711 uncorrected / 0.715 corrected) and mseq with its MART regression model trained on the 100 most highly expressed transcripts (expected R2 0.730 uncorrected / 0.755 corrected), writing all R2 values to a CSV such as comparison_r2_values.csv or qpcr_correlation_results.csv.

## Related tools

- **Cufflinks** (Estimate transcript expression from mapped MAQC HBR alignments both with and without fragment-bias correction enabled)
- **Genominator** (Count-based baseline method run with bias parameters learned per chromosome to produce uncorrected/corrected comparison R2 values)
- **mseq** (MART-regression-based baseline whose model is trained on the 100 most highly expressed transcripts after Bowtie remapping)
- **Bowtie** (Remap MAQC HBR reads to the transcriptome before mseq-based expression estimation)
- **Python** (Converts standard SAM and FASTA input into the mseq input format and computes R2/permutation-test statistics)
- **NanoString** (Supplementary expression measurements included alongside the qRT-PCR comparison data)
- **eXpress-D** (Distributed ambiguous fragment assignment using eXpress and Spark, for likelihood-based simultaneous bias and expression estimation at scale) — github.com/adarob/eXpress-d

## Examples

```
python scripts/convert_sam.py --sam maqc_hbr.sam --fasta transcripts.fa > mseq_input.txt && bowtie -v 2 transcripts.idx maqc_hbr.fq mseq.sam
```

## Evaluation signals

- Bias-corrected R2 must exceed uncorrected R2 for each method (e.g., Cufflinks 0.807 > 0.753 on the 907 uniquely probed MAQC HBR transcripts)
- Permutation-test P-value for the Cufflinks improvement should equal the reported 0.0007
- Reproduced baseline R2 values should match reported values within a small numeric tolerance: Genominator 0.711/0.715 and mseq 0.730/0.755
- All four cross-method R2 values are written to comparison_r2_values.csv and the three Cufflinks values (uncorrected R2, corrected R2, permutation P) to qpcr_correlation_results.csv
- The uncorrected and corrected correlations must be computed over the exact same transcript set (the 907 uniquely probed transcripts)

## Limitations

- The validation set is restricted to uniquely probed transcripts (907 on MAQC HBR), so results do not generalize to the full transcriptome or multi-probed loci
- Baseline bias corrections gave only marginal gains for some methods (Genominator 0.711 -> 0.715), so a tiny corrected-uncorrected gap can be method-dependent rather than a pipeline failure
- The mseq baseline requires training its MART regression on the 100 most highly expressed transcripts, which limits applicability to datasets with sufficiently many highly expressed transcripts

## Evidence

- [results] Bias correction improved the correlation of Cufflinks expression estimates with qRT-PCR on the MAQC HBR dataset, raising R2 from 0.753 (uncorrected) to 0.807 (corrected), a highly significant improvement (permutation-test P = 0.0007).: [evidence span withheld — anchor status: not_found]
- [results] the Genominator-based method of [8] yielded R2 = 0.711 before bias correction and R2 = 0.715 after bias correction.: [evidence span withheld — anchor status: not_found]
- [supplementary] retain only the 907 transcripts whose TaqMan probes map uniquely: [evidence span withheld — anchor status: not_found]
- [supplementary] Python script that converts standard SAM and FASTA input into the mseq input format.: "Python script that converts standard SAM and FASTA input into the mseq input format."
- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [readme] Distributed ambiguous fragment assignment using eXpress and Spark.: "Distributed ambiguous fragment assignment using eXpress and Spark."
