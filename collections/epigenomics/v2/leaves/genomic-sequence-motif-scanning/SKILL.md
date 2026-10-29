---
name: genomic-sequence-motif-scanning
description: Use when you have scored sequences (e.g. ATAC-seq log2 fold-change scores
  on ±100 bp windows around a GATA1 motif) in FASTA with score headers and need to
  locate occurrences of known motifs from a fixed JASPAR/HOMER library for positional
  enrichment analysis.
when_to_use_negative:
- You need de novo motif discovery rather than scanning a fixed motif library — the
  article states 'MEPP currently functions with a fixed motif library' and defers
  de novo recognition/assembly to future work
- You only need motif presence/absence enrichment independent of position within sequences
  (e.g. a standard ZOOPs-style MEA such as HOMER's top-bin analysis) rather than positional
  profiling
- Sequences are not centered on a biologically meaningful anchor point (TSS or motif),
  since positional profiles are only interpretable relative to such anchors
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: MEPP
  role: Scans scored, anchor-centered sequences with MOODS against a JASPAR-format
    motif library and converts motif match positions into positional enrichment profiles
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installs the MEPP package used for motif scanning ('pip install mepp' or from
    the GitHub repo)
  repo: https://github.com/npdeloss/mepp
- name: HOMER
  role: Source of the motif matrix files bundled in data/homer.motifs.txt used as
    the scanning library; also used as a ZOOPs-model comparison MEA tool
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:b38b7240f1bdd525342be0fde81209d8f231849075a0796f7f5b0abb570f4e95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - CentriMo
  - HOMER
  - MEPP
  - TFEA
  - pip
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# genomic-sequence-motif-scanning

## Summary

Scans scored DNA sequences centered on anchor points (e.g. GATA1 motifs or TSS) with fixed motif position frequency matrices, using MEPP's MOODS-based thresholded scanning (default p-value 0.0001, pseudocount 0.0001) to detect motif occurrences whose positions are then correlated with sequence-level biological scores.

## When to use

Use when you have scored sequences (e.g. ATAC-seq log2 fold-change scores on ±100 bp windows around a GATA1 motif) in FASTA with score headers and need to locate occurrences of known motifs from a fixed JASPAR/HOMER library for positional enrichment analysis.

## When NOT to use

- You need de novo motif discovery rather than scanning a fixed motif library — the article states 'MEPP currently functions with a fixed motif library' and defers de novo recognition/assembly to future work
- You only need motif presence/absence enrichment independent of position within sequences (e.g. a standard ZOOPs-style MEA such as HOMER's top-bin analysis) rather than positional profiling
- Sequences are not centered on a biologically meaningful anchor point (TSS or motif), since positional profiles are only interpretable relative to such anchors

## Inputs

- Scored FASTA file with headers of the form '>sequence_name sequence_score' (e.g. ATAC-seq log2 fold-change accessibility per ±100 bp GATA1-centered window)
- Motif matrices file in JASPAR format (or bundled data/homer.motifs.txt / data/ohler_motifs.txt)
- Background nucleotide composition (default 0.25 0.25 0.25 0.25)

## Outputs

- Per-motif positional match scores/heatmaps along the scored sequences relative to the anchor point
- MEPP positional enrichment profile plots (png/svg) showing where motif presence correlates with the sequence score
- Clustering table of positional correlations with adjusted p-values (multiple testing via statsmodels, default fdr_by, alpha 0.01)

## How to apply

Format sequences as a scored FASTA where each header is of the form '>sequence_name sequence_score' (e.g. log2 fold-change accessibility of HCT116-vs-K562 ATAC-seq for a ±100 bp window centered on a GATA1 motif). Supply a motif matrices file in JASPAR format — the mepp repo bundles data/homer.motifs.txt and a reduced-redundancy clustered version, plus data/ohler_motifs.txt for Drosophila core promoter motifs. MEPP scans each sequence with MOODS, setting the match threshold from a p-value of 0.0001 with pseudocount 0.0001 and background 0.25 0.25 0.25 0.25, and 'blurs' matches by a default margin of 2 bases for smoothing. Choose motif orientations via --orientations (default '+,+/-'), optionally filter repeat-laden sequences with --dgt (percentage of degenerate bases tolerated before rejection). The scan produces per-position motif match scores (heatmaps) that feed the positional enrichment profile correlating motif position with the sequence score, controlling for lower-order nucleotide bias.

## Related tools

- **MEPP** (Scans scored, anchor-centered sequences with MOODS against a JASPAR-format motif library and converts motif match positions into positional enrichment profiles) — https://github.com/npdeloss/mepp
- **pip** (Installs the MEPP package used for motif scanning ('pip install mepp' or from the GitHub repo)) — https://github.com/npdeloss/mepp
- **HOMER** (Source of the motif matrix files bundled in data/homer.motifs.txt used as the scanning library; also used as a ZOOPs-model comparison MEA tool)

## Examples

```
mepp --fa gata1_centered_scored.fa --motifs data/homer.motifs.txt --out mepp_output --pval 0.0001 --pcount 0.0001 --orientations +,+/-
```

## Evaluation signals

- Every input FASTA header parses as '>sequence_name sequence_score'; sequences with more degenerate (non-ACGT) bases than the --dgt percentage are rejected
- Motif match thresholds derive from MOODS with the specified --pval (default 0.0001) and --pcount (default 0.0001); scanning covers both orientations when --orientations '+,/-' is set
- For the K562/HCT116 GATA1-anchored data, the scan should reveal the SCL (TAL1) motif match positions enriched ~12 bp upstream of the GATA1 anchor in the resulting positional profile
- Positional correlation p-values in the clustering table are multiple-testing adjusted (default fdr_by at alpha 0.01) and non-degenerate
- MEPP plots are generated with default 95.0% confidence intervals from 1000 permutations; profiles cluster with default average linkage and correlation metric

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified for MEPP and no changelog is available in the repository
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage; GPU jobs must be tuned to avoid crowding each other out of GPU memory, or GPU use disabled with --nogpu
- Scanning requires scores attached per sequence; unscored FASTA cannot produce the score-dependent positional profile

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [methods] $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [readme] P-value for setting motif match threshold via MOODS. Default: 0.0001: "--pval FLOAT                    P-value for setting motif match threshold
                                      via MOODS. Default: 0.0001"
- [readme] sequence headers are of the form: ">sequence_name sequence_score": "--fa TEXT                       Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name"
- [discussion] MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work: "MEPP currently functions with a fixed motif library. However, the underlying convolutional network architecture lends itself easily to future work for recognizing and assembling de novo motifs based"
