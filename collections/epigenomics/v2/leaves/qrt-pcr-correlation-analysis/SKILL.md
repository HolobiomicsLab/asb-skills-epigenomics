---
name: qrt-pcr-correlation-analysis
description: Use when expression estimates from an RNA-Seq method (uncorrected or
  bias-corrected) must be benchmarked against independently performed qRT-PCR measurements
  to quantify whether bias correction improved accuracy.
when_to_use_negative:
- No independent qRT-PCR (or equivalent) reference measurements exist for the sample,
  so correlation with a gold standard cannot be computed
- The goal is internal assessment of expression estimates without bias correction
  applied — a single uncorrected run leaves no corrected/uncorrected contrast to evaluate
- TaqMan probes do not map uniquely to transcripts, since the benchmark restricts
  to uniquely probed transcripts (907 on MAQC HBR)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: Cufflinks
  role: Estimator run twice on the same alignments, without and with fragment-bias
    correction, to obtain the two expression-estimate sets correlated against qRT-PCR
- name: mseq
  role: MART regression baseline trained on the 100 most highly expressed transcripts
    after Bowtie remapping, producing uncorrected and corrected estimates for comparison
- name: Genominator
  role: Count-based baseline method run with bias parameters learned per chromosome,
    evaluated uncorrected (R2 = 0.711) and corrected (R2 = 0.715)
- name: Bowtie
  role: Remaps MAQC HBR reads to the transcriptome to generate alignments for the
    mseq baseline
- name: Python
  role: Runs the SAM/FASTA-to-mseq conversion script and computes R2 correlations
    and permutation tests, writing results to CSV
- name: eXpress-D
  role: Likelihood-based simultaneous estimation of bias parameters and expression
    levels whose corrected estimates are compared against qRT-PCR
  repo: https://github.com/adarob/express-d/wiki
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1186/gb-2011-12-3-r22
    title: Improving RNA-Seq expression estimates by correcting for fragment bias
  source_content_hash: sha256:f96024d9803a10819763ffdb241ba4723c5f2cfccaa360ceab6e5a8e8a537d69
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
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# qrt-pcr-correlation-analysis

## Summary

Validates RNA-Seq expression estimates (raw and bias-corrected) by computing squared Pearson correlation (R2) against independently measured qRT-PCR reference values, as done on the MAQC Human Brain Reference benchmark where Cufflinks improved from R2 = 0.753 to 0.807 and mseq/Genominator baselines reached 0.730/0.755 and 0.711/0.715.

## When to use

Use when expression estimates from an RNA-Seq method (uncorrected or bias-corrected) must be benchmarked against independently performed qRT-PCR measurements to quantify whether bias correction improved accuracy.

## When NOT to use

- No independent qRT-PCR (or equivalent) reference measurements exist for the sample, so correlation with a gold standard cannot be computed
- The goal is internal assessment of expression estimates without bias correction applied — a single uncorrected run leaves no corrected/uncorrected contrast to evaluate
- TaqMan probes do not map uniquely to transcripts, since the benchmark restricts to uniquely probed transcripts (907 on MAQC HBR)

## Inputs

- Mapped RNA-Seq alignments (SAM) for the benchmark sample, e.g., MAQC HBR SRA012427 reads mapped with Bowtie
- Uncorrected transcript expression estimates from the estimator
- Bias-corrected transcript expression estimates from the same estimator
- Independently measured qRT-PCR reference expression values with transcript/probe identifiers
- Filter list or mapping of uniquely mapping TaqMan probes (e.g., 907 uniquely probed transcripts)

## Outputs

- Uncorrected R2 value between qRT-PCR and expression estimates
- Corrected R2 value between qRT-PCR and bias-corrected estimates
- Permutation-test P-value for the R2 improvement (e.g., 0.0007)
- CSV of comparison results (qpcr_correlation_results.csv or comparison_r2_values.csv with four R2 values for Genominator/mseq)

## How to apply

Map RNA-Seq reads (e.g., SRA012427 MAQC HBR alignments) to the transcriptome, then run the estimator twice: once without bias correction and once with fragment-bias correction enabled (e.g., Cufflinks, or mseq/Genominator with bias learned per chromosome). Load the independently performed qRT-PCR measurements and filter to the transcripts whose TaqMan probes map uniquely (907 transcripts on the MAQC HBR benchmark). Compute the squared Pearson correlation (R2) between qRT-PCR values and both the uncorrected and corrected estimates over this common transcript set. Assess significance of the R2 improvement with a permutation test comparing corrected and uncorrected correlations (the Cufflinks comparison reported P = 0.0007), and write the results (e.g., uncorrected R2, corrected R2, P-value) to a CSV such as qpcr_correlation_results.csv or comparison_r2_values.csv.

## Related tools

- **Cufflinks** (Estimator run twice on the same alignments, without and with fragment-bias correction, to obtain the two expression-estimate sets correlated against qRT-PCR)
- **mseq** (MART regression baseline trained on the 100 most highly expressed transcripts after Bowtie remapping, producing uncorrected and corrected estimates for comparison)
- **Genominator** (Count-based baseline method run with bias parameters learned per chromosome, evaluated uncorrected (R2 = 0.711) and corrected (R2 = 0.715))
- **Bowtie** (Remaps MAQC HBR reads to the transcriptome to generate alignments for the mseq baseline)
- **Python** (Runs the SAM/FASTA-to-mseq conversion script and computes R2 correlations and permutation tests, writing results to CSV)
- **eXpress-D** (Likelihood-based simultaneous estimation of bias parameters and expression levels whose corrected estimates are compared against qRT-PCR) — https://github.com/adarob/express-d/wiki

## Examples

```
python compute_r2.py --uncorrected cufflinks_uncorrected.csv --corrected cufflinks_corrected.csv --qpcr taqman_qpcr.csv --filter unique_probes.tsv --out qpcr_correlation_results.csv
```

## Evaluation signals

- Bias-corrected R2 must exceed uncorrected R2 (e.g., Cufflinks 0.807 > 0.753; mseq 0.755 > 0.730; Genominator 0.715 > 0.711)
- Reproduced R2 values should match reported values within a small numeric tolerance (Cufflinks 0.753/0.807; Genominator 0.711/0.715; mseq 0.730/0.755)
- Permutation-test P-value for the improvement should equal the reported value (P = 0.0007 for Cufflinks)
- Correlation must be computed over the same filtered transcript set for both runs — only transcripts with uniquely mapping TaqMan probes (907) are retained
- Output CSV must contain the uncorrected R2, corrected R2, and permutation-test P-value with matching schema

## Limitations

- Validation depends on availability of independently performed qRT-PCR measurements for the same sample (e.g., MAQC HBR); datasets without such a standard cannot be scored this way
- Only transcripts with uniquely mapping probes are usable, so the comparison covers a restricted transcript subset rather than the whole transcriptome
- Reported R2 gains differ by method and can be small (Genominator improved only 0.711 to 0.715), so statistical testing (permutation test) is needed rather than eyeballing the delta

## Evidence

- [abstract] We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR: "We find improvements in expression estimates as measured by correlation with independently performed qRT-PCR"
- [intro] Benchmark comparison against prior methods on the same data: "Using the same data, we also show that our method improves on the approaches of [8,10]."
- [intro] Simultaneous estimation of bias and expression underlies the corrected estimates being validated: "Our main result is the development of a likelihood based approach for simultaneous estimation of bias parameters and expression levels using the likelihood framework of [6]."
- [intro] Cross-protocol validation including NSR-primed MAQC HBR against qRT-PCR: "One of our main findings is that bias correction improves the correlation of expression estimates obtained from sequence data generated using different sample preparations and different sequencing"
- [supplementary] NanoString expression measurements available as supplementary validation data: "including the NanoString expression measurements"
