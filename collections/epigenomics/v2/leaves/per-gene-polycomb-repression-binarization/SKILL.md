---
name: per-gene-polycomb-repression-binarization
description: Use when a cell type-specific binary annotation of polycomb-repressed
  genes is needed from H3K27me3 ChIP-seq, e.g. as an epigenetic-state input feature
  for SV pathogenicity prediction in POSTRE-style analyses.
when_to_use_negative:
- Input is already a per-gene repression or epigenetic-state feature table — re-deriving
  the score from ChIP-seq would be redundant
- The histone mark produces narrow punctate peaks rather than broad polycomb domains,
  so a broad-mode peak-size cutoff is not the right model
- A continuous quantitative H3K27me3 occupancy measure is required — the binarization
  to 0/1 discards signal magnitude
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: MACS2
  role: Calls broad H3K27me3 peaks in broad mode using the matched input control per
    dataset
- name: bedtools
  role: Merges called broad peaks within each dataset into a non-redundant peak catalog
    before cutoff filtering
- name: findiplist (R inflection package)
  role: Derives the dataset-specific peak-size cutoff from the upper limit value of
    the inflection-point analysis of the merged-peak length distribution
- name: POSTRE
  role: Downstream framework that consumes the binary polycombScore as a per-cell-type
    gene annotation for SV pathogenicity prediction
  repo: https://github.com/vicsanga/Postre
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:4eac93968aff542b2f2519f92e8dd62490356bc26e22450ecf192f0259942c58
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
  - POSTRE
  - R
  - bedtools
  - findiplist (R inflection package)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# per-gene-polycomb-repression-binarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A POSTRE-derived skill that converts H3K27me3 ChIP-seq broad peaks into a binary per-gene polycombScore, quantifying the breadth of polycomb domains in gene promoters per cell type. It derives a dataset-specific peak-size cutoff via findiplist() inflection-point analysis instead of using a fixed threshold.

## When to use

Use when a cell type-specific binary annotation of polycomb-repressed genes is needed from H3K27me3 ChIP-seq, e.g. as an epigenetic-state input feature for SV pathogenicity prediction in POSTRE-style analyses.

## When NOT to use

- Input is already a per-gene repression or epigenetic-state feature table — re-deriving the score from ChIP-seq would be redundant
- The histone mark produces narrow punctate peaks rather than broad polycomb domains, so a broad-mode peak-size cutoff is not the right model
- A continuous quantitative H3K27me3 occupancy measure is required — the binarization to 0/1 discards signal magnitude

## Inputs

- H3K27me3 ChIP-seq FASTQ files from GEO/SRA (e.g. SRR067372, SRR6418921, SRR4032228)
- Matched input control FASTQ files for each ChIP-seq dataset (e.g. SRR067371, SRR6418919, SRR4032231)
- hg19 gene body annotations

## Outputs

- polycombScore.tsv — tab-separated plain text file with a per-gene binary 0/1 polycombScore
- Per-dataset derived peak-size cutoff in bp (e.g. 6689 bp ESC, 6719 bp NCC, 8020 bp cardiomyocytes)

## How to apply

Download H3K27me3 ChIP-seq and matched input control FASTQs (e.g. GSE24447 SRR067372/SRR067371 for ESC, GSE108518 SRR6418921/SRR6418919 for NCC, GSE85628 SRR4032228/SRR4032231 for cardiomyocytes). Call broad H3K27me3 peaks with MACS2 in broad mode, using the matched input control for each dataset and hg19 coordinates, since H3K27me3 domains are broad rather than punctate. Merge called peaks within each dataset with bedtools merge to obtain a non-redundant peak catalog. Compute the merged-peak length distribution and derive the size cutoff with findiplist() from the R inflection package (upper limit value; expected ~6689 bp ESC, ~6719 bp NCC, ~8020 bp cardiomyocytes) — the cutoff is cell-type-specific because polycomb domain breadth differs between cell types. Overlap merged peaks exceeding the dataset-specific cutoff with hg19 gene bodies and binarize each gene to 0/1, then write the table as a tab-separated plain text file named polycombScore.tsv.

## Related tools

- **MACS2** (Calls broad H3K27me3 peaks in broad mode using the matched input control per dataset)
- **bedtools** (Merges called broad peaks within each dataset into a non-redundant peak catalog before cutoff filtering)
- **findiplist (R inflection package)** (Derives the dataset-specific peak-size cutoff from the upper limit value of the inflection-point analysis of the merged-peak length distribution)
- **POSTRE** (Downstream framework that consumes the binary polycombScore as a per-cell-type gene annotation for SV pathogenicity prediction) — https://github.com/vicsanga/Postre

## Examples

```
bedtools merge -i H3K27me3_broadPeaks.bed > merged_peaks.bed
```

## Evaluation signals

- The findiplist() derived cutoffs fall near 6689 bp for GSE24447, 6719 bp for GSE108518 and 8020 bp for GSE85628
- Every gene row in polycombScore.tsv contains a binary 0/1 score
- Peak catalog is non-redundant after bedtools merge within each dataset
- Matched input control was used for MACS2 broad-mode calling in each dataset, with hg19 coordinates throughout
- Only merged peaks exceeding the dataset-specific cutoff were overlapped with hg19 gene bodies

## Limitations

- The peak-size cutoff is dataset-specific and must be re-derived per cell type; reusing one cell type's cutoff (e.g. 8020 bp) on another biases gene assignment
- The score is binary, so graded differences in H3K27me3 domain breadth or intensity are lost
- Coordinates are hg19; lifting to other genome builds requires additional processing
- Bulk cell-type genomic data may obscure subpopulations, and lack of data for the appropriate developmental stage can affect identification of pathologically relevant genes

## Evidence

- [results] POSTRE derives a per-cell-type H3K27me3 peak-size cutoff from the upper limit value of the findiplist() inflection-point analysis: [evidence span withheld — anchor status: not_found]
- [methods] Call broad H3K27me3 peaks with MACS2 in broad mode, using the matched input control for each dataset and hg19 coordinates.: [evidence span withheld — anchor status: not_found]
- [methods] Compute the merged-peak length distribution and derive the size cutoff per dataset with findiplist(): [evidence span withheld — anchor status: not_found]
- [methods] Overlap merged peaks exceeding the dataset-specific cutoff with hg19 gene bodies and binarize each gene to 0/1.: [evidence span withheld — anchor status: not_found]
