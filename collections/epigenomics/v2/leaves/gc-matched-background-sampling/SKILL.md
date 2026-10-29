---
name: gc-matched-background-sampling
description: Use when TFBS motif instances from CNN motif detectors must be retained
  by statistical significance rather than a subjective signal cutoff — i.e., when
  per-motif occurrence probabilities from GC-matched 101-bp background bins are needed
  to compute binomial P-values.
when_to_use_negative:
- 'Peak sequences are not 101 bp (e.g. 1001-bp GCNN runs with peak_flank 500): the
  supplied 101-bp background bins would not match peak length or GC distribution,
  so a newly matched background is required.'
- The goal is motif-to-database similarity scoring (TOMTOM against documented JASPAR/TRANSFAC
  motifs) rather than statistical significance of TFBS counts against genomic background.
- Working outside the supplied human ENCODE/GRCh37.p13 setup with no ability to construct
  a GC-matched background of matching length and composition.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: DESSO
  role: Ships the GC-matched background set (encode_101_background unpacked into data/)
    and the predict.py cutoff-scan motif-instance outputs from which per-motif occurrence
    probabilities and binomial P-values are computed
  repo: https://github.com/OSU-BMBL/DESSO
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkz672
    title: Prediction of regulatory motifs from human Chip-sequencing data using a
      deep learning framework
  source_content_hash: sha256:68c92a006fe5cb5cd8aa258d75744fb691ee83e7df54385de58a6f12acf86eea
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OSU-BMBL/DESSO
  repo_url: https://github.com/OSU-BMBL/DESSO
  tools:
  - Basset
  - DESSO
  - DeepBind
  - KMAC
  - TOMTOM
  - gkm-SVM
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gc-matched-background-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs DESSO's GC-matched background sampling, in which 500,000 GC-matched 101-bp genomic bins (encode_101_background) supply the per-motif occurrence probabilities for the binomial P-value model that replaces subjective motif signal cutoffs in TFBS identification.

## When to use

Use when TFBS motif instances from CNN motif detectors must be retained by statistical significance rather than a subjective signal cutoff — i.e., when per-motif occurrence probabilities from GC-matched 101-bp background bins are needed to compute binomial P-values.

## When NOT to use

- Peak sequences are not 101 bp (e.g. 1001-bp GCNN runs with peak_flank 500): the supplied 101-bp background bins would not match peak length or GC distribution, so a newly matched background is required.
- The goal is motif-to-database similarity scoring (TOMTOM against documented JASPAR/TRANSFAC motifs) rather than statistical significance of TFBS counts against genomic background.
- Working outside the supplied human ENCODE/GRCh37.p13 setup with no ability to construct a GC-matched background of matching length and composition.

## Inputs

- encode_101_background: GC-matched background set of 500,000 101-bp genomic bins (zip unpacked into data/)
- DESSO predict.py motif-instance candidate sets Omega(M, lambda) produced by the cutoff scan --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
- 101-bp peak sequences extracted with peak_flank 50 from the 690 human ENCODE ChIP-seq datasets (data/encode_101, GRCh37.p13 genome)

## Outputs

- Per-motif occurrence probabilities estimated from the 500,000 GC-matched background bins
- Per-motif, per-cutoff binomial P-value table with the optimal (minimum-P) cutoff and pass/fail retention decisions (binomial_pvalues.tsv)
- pvalue_validation_report.tsv recording deviations against an independent binomial-CDF reference implementation

## How to apply

Download and unzip encode_101_background (500,000 GC-matched 101-bp bins) into data/ so the background bins match the length of the ChIP-seq peak sequences (peak_flank 50 gives 2*50+1 = 101 bp) and are comparable in GC content. Parse the DESSO predict.py motif-instance outputs generated with the cutoff scan --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03, and for each motif M and cutoff lambda build the candidate set Omega(M, lambda) of peak sequences whose predicted binding sites pass that cutoff. Estimate each motif's occurrence probability from the 500,000 background bins and compute binomial P-values (formula 5) for every Omega(M, lambda) with an exact binomial CDF implementation — this is the rationale of the method: the optimal TFBS set per motif detector is the candidate set with the minimum binomial P-value. Apply the retention filters (P < 1e-4, more than 5 motif instances, at least 3 positions with information content > 1) and write the per-motif, per-cutoff P-value table with pass/fail decisions (binomial_pvalues.tsv). Validate computed P-values against an independent binomial-CDF reference implementation for fixed lambda values on the supplied background bin set, recording deviations in pvalue_validation_report.tsv.

## Related tools

- **DESSO** (Ships the GC-matched background set (encode_101_background unpacked into data/) and the predict.py cutoff-scan motif-instance outputs from which per-motif occurrence probabilities and binomial P-values are computed) — https://github.com/OSU-BMBL/DESSO

## Examples

```
python predict.py --start_index 0 --end_index 1 --peak_flank 50 --network CNN --feature_format Seq --start_cutoff 0.01 --end_cutoff 1 --step_cutoff 0.03
```

## Evaluation signals

- Background set loads as 500,000 GC-matched 101-bp bins, matching the peak length (2*50+1 = 101 bp) of the encode_101 peak sequences.
- Estimated per-motif occurrence probabilities fall in [0, 1] and are computed per motif from the same background bin set.
- Binomial P-values for fixed cutoff lambda reproduce an independent exact binomial-CDF reference implementation within numerical tolerance, with deviations logged in pvalue_validation_report.tsv.
- The selected TFBS set per motif corresponds to the minimum binomial P-value over the scanned cutoff grid lambda in {0.01, 0.04, ..., 1} (start_cutoff 0.01, step_cutoff 0.03).
- Retained motifs satisfy the filters: P < 1e-4, more than 5 motif instances, and at least 3 positions with information content > 1.

## Limitations

- The supplied encode_101_background is fixed for the human ENCODE setup (GRCh37.p13, 101-bp bins); other genomes or peak lengths need a newly GC-matched background.
- The GitHub repository only contains wgEncodeEH002288-related data due to file-size limits; the full ~5.9GB code+data package is required to cover all 690 datasets.
- The binomial treatment, like the article's noted position-independence assumption of DL-based methods, ignores nucleotide dependencies, which may underestimate TF-DNA binding complexity.

## Evidence
