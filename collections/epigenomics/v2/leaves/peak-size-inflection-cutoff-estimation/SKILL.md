---
name: peak-size-inflection-cutoff-estimation
description: Use when calling broad histone-mark peaks (H3K27me3) and needing a data-driven,
  cell-type-specific peak-size cutoff to decide which merged peaks count as Polycomb
  domains over genes.
when_to_use_negative:
- Input is narrow-mark ChIP-seq (e.g. TF or H3K4me1/H3K27ac peaks) where a domain-size
  inflection cutoff is not meaningful.
- The cell type of interest lacks public H3K27me3 ChIP-seq with a matched input control,
  so a dataset-specific cutoff cannot be derived.
- Only fixed promoter-window statistics are needed; this skill binarizes gene bodies
  from large Polycomb domains, not promoter coverage per se.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: MACS2
  role: Calls broad H3K27me3 peaks in broad mode using the matched input control for
    each dataset
- name: bedtools
  role: Merges called broad peaks within each dataset to obtain a non-redundant peak
    catalog
- name: findiplist (R inflection package)
  role: Computes the inflection point of the merged-peak length distribution; its
    upper limit value defines the per-dataset peak-size cutoff
- name: R
  role: Runtime environment (>=3.5.0) for POSTRE and the findiplist() cutoff derivation
  repo: https://github.com/vicsanga/Postre
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:58f298e4852b90e5db793f0f992c2057e022e13f8e3e8331c874624a328e7fbd
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  repo_url: https://github.com/vicsanga/Postre
  tools:
  - MACS2
  - R
  - bedtools
  - findiplist (R inflection package)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# peak-size-inflection-cutoff-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives a per-dataset size cutoff for broad H3K27me3 ChIP-seq peaks using the findiplist() inflection-point analysis from the R inflection package, so that only peaks as large as Polycomb domains (e.g. ~6689 bp in ESC, ~6719 bp in NCC, ~8020 bp in cardiomyocytes) define the binary polycombScore metric in POSTRE.

## When to use

Use when calling broad histone-mark peaks (H3K27me3) and needing a data-driven, cell-type-specific peak-size cutoff to decide which merged peaks count as Polycomb domains over genes.

## When NOT to use

- Input is narrow-mark ChIP-seq (e.g. TF or H3K4me1/H3K27ac peaks) where a domain-size inflection cutoff is not meaningful.
- The cell type of interest lacks public H3K27me3 ChIP-seq with a matched input control, so a dataset-specific cutoff cannot be derived.
- Only fixed promoter-window statistics are needed; this skill binarizes gene bodies from large Polycomb domains, not promoter coverage per se.

## Inputs

- H3K27me3 ChIP-seq FASTQ files from GEO/SRA (GSE24447, GSE108518, GSE85628)
- Matched input-control FASTQ files per dataset
- hg19 gene body annotations
- Merged broad-peak catalog per cell type (BED)

## Outputs

- Per-dataset H3K27me3 peak-size cutoff in bp (upper limit of findiplist() inflection-point analysis)
- Per-gene binary polycombScore table written as polycombScore.tsv (tab-separated, 0/1 values)

## How to apply

Download H3K27me3 ChIP-seq and matched input control FASTQs (e.g. GSE24447 SRR067372/SRR067371 for ESC, GSE108518 SRR6418921/SRR6418919 for NCC, GSE85628 SRR4032228/SRR4032231 for cardiomyocytes), then call broad peaks with MACS2 in broad mode against the matched input control using hg19 coordinates. Merge called peaks within each dataset with bedtools merge to obtain a non-redundant peak catalog. Compute the merged-peak length distribution and apply findiplist() from the R inflection package, taking the upper limit value of the inflection-point analysis as the dataset-specific size cutoff (expected ~6689 bp for GSE24447, ~6719 bp for GSE108518, ~8020 bp for GSE85628). Overlap only merged peaks exceeding this cutoff with hg19 gene bodies and binarize each gene to 0/1, writing the per-gene polycombScore table as a tab-separated plain text file named polycombScore.tsv. The rationale is that genuine Polycomb domains are very large H3K27me3 blocks, so a length inflection separates domain-scale peaks from short noise peaks.

## Related tools

- **MACS2** (Calls broad H3K27me3 peaks in broad mode using the matched input control for each dataset)
- **bedtools** (Merges called broad peaks within each dataset to obtain a non-redundant peak catalog)
- **findiplist (R inflection package)** (Computes the inflection point of the merged-peak length distribution; its upper limit value defines the per-dataset peak-size cutoff)
- **R** (Runtime environment (>=3.5.0) for POSTRE and the findiplist() cutoff derivation) — https://github.com/vicsanga/Postre

## Examples

```
macs2 callpeak -t H3K27me3.bam -c input.bam --broad -g hs -n ESC_H3K27me3 && bedtools merge -i ESC_H3K27me3_broad Peaks.bed > merged_peaks.bed && Rscript -e 'findiplist(x, y, index="all")'
```

## Evaluation signals

- The findiplist()-derived cutoffs fall near the expected values: ~6689 bp for GSE24447 (ESC), ~6719 bp for GSE108518 (NCC), ~8020 bp for GSE85628 (cardiomyocytes).
- Every gene row in polycombScore.tsv contains a binary 0/1 score with no intermediate values.
- bedtools merge produces a non-redundant peak catalog (no overlapping intervals within a dataset) before length statistics are computed.
- Only merged peaks longer than the dataset-specific cutoff contribute positive polycombScore assignments.
- All genomic interval operations are performed in hg19 coordinates, matching the POSTRE breakpoint coordinate system.

## Limitations

- The cutoff is dataset-specific and re-derived per cell type; a cutoff from one H3K27me3 dataset should not be reused for another without recomputing the length distribution.
- Only peaks exceeding the cutoff are counted as Polycomb domains, so shorter H3K27me3 signals below the inflection are discarded and may miss weaker domain coverage.
- Cutoffs are tied to the bulk ChIP-seq datasets used by POSTRE (ESC, NCC, cardiomyocytes on hg19); applying the metric to other marks, assemblies, or cell types requires re-deriving the inflection.

## Evidence

- [results] POSTRE derives a per-cell-type H3K27me3 peak-size cutoff from the upper limit value of the findiplist() inflection-point analysis: [evidence span withheld — anchor status: not_found]
- [methods] Call broad H3K27me3 peaks with MACS2 in broad mode, using the matched input control for each dataset and hg19 coordinates.: [evidence span withheld — anchor status: not_found]
- [methods] Merge the called broad peaks within each dataset using bedtools merge to obtain a non-redundant peak catalog.: [evidence span withheld — anchor status: not_found]
- [methods] Overlap merged peaks exceeding the dataset-specific cutoff with hg19 gene bodies and binarize each gene to 0/1.: [evidence span withheld — anchor status: not_found]
- [results] Validation: the findiplist() cutoffs fall near 6689/6719/8020 bp and every gene row in polycombScore.tsv contains a binary 0/1 score.: [evidence span withheld — anchor status: not_found]
