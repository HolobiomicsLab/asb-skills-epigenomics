---
name: h3k27me3-broad-peak-calling
description: Use when building a cell type-specific H3K27me3/polycomb domain annotation
  from public ChIP-seq (FASTQ from GEO/SRA), especially to compute POSTRE's binary
  per-gene polycombScore.
when_to_use_negative:
- The aim is narrow/point-source histone marks (e.g. H3K4me3 or H3K27ac enhancer/promoter
  peaks) — MACS2 broad mode and the domain-size cutoff logic are specific to broad
  polycomb domains
- The target cell type lacks publicly available H3K27me3 ChIP-seq with a matched input
  control; POSTRE itself notes that lack of data for appropriate developmental stages
  can affect identification of relevant genes
- The analysis already has a per-gene polycombScore table (e.g. from POSTRE test files);
  re-deriving it duplicates the workflow rather than consuming its output
edam_operation: http://edamontology.org/operation_3222
tools:
- name: MACS2
  role: Calls broad H3K27me3 peaks in broad mode using matched input controls
- name: bedtools
  role: Merges overlapping broad peaks within each dataset into a non-redundant peak
    catalog and overlaps peaks with hg19 gene bodies
- name: findiplist (R inflection package)
  role: Inflection-point analysis of the merged-peak length distribution to derive
    the per-dataset peak-size cutoff
- name: R
  role: Runtime environment (>=3.5.0) for the cutoff computation and polycombScore
    binarization
  repo: https://github.com/vicsanga/Postre
- name: POSTRE
  role: Consumes the resulting polycombScore.tsv as part of its cell type-specific
    regulatory annotations
  repo: https://github.com/vicsanga/Postre
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:4eb16c4afc97b0a54ce15aa06b78bcef58f0b63e74dd1b89dfa374a022e4e355
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# h3k27me3-broad-peak-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the polycombScore module used by POSTRE by calling broad H3K27me3 ChIP-seq peaks with MACS2, deriving a per-cell-type peak-size cutoff via findiplist() inflection-point analysis (6689 bp ESC, 6719 bp NCC, 8020 bp cardiomyocytes), and binarizing genes overlapping large polycomb domains.

## When to use

Use when building a cell type-specific H3K27me3/polycomb domain annotation from public ChIP-seq (FASTQ from GEO/SRA), especially to compute POSTRE's binary per-gene polycombScore.

## When NOT to use

- The aim is narrow/point-source histone marks (e.g. H3K4me3 or H3K27ac enhancer/promoter peaks) — MACS2 broad mode and the domain-size cutoff logic are specific to broad polycomb domains
- The target cell type lacks publicly available H3K27me3 ChIP-seq with a matched input control; POSTRE itself notes that lack of data for appropriate developmental stages can affect identification of relevant genes
- The analysis already has a per-gene polycombScore table (e.g. from POSTRE test files); re-deriving it duplicates the workflow rather than consuming its output

## Inputs

- H3K27me3 ChIP-seq FASTQ files from GEO/SRA (e.g. SRR067372, SRR6418921, SRR4032228)
- Matched input control FASTQ files for each dataset (e.g. SRR067371, SRR6418919, SRR4032231)
- hg19 gene body coordinates

## Outputs

- Non-redundant catalog of merged broad H3K27me3 peaks per cell type
- Per-dataset peak-size cutoff in bp (6689 bp ESC, 6719 bp NCC, 8020 bp cardiomyocytes)
- polycombScore.tsv: tab-separated plain text table of per-gene binary 0/1 polycombScore

## How to apply

Download H3K27me3 ChIP-seq and matched input control FASTQs (e.g. GSE24447 SRR067372/SRR067371, GSE108518 SRR6418921/SRR6418919, GSE85628 SRR4032228/SRR4032231) and call peaks with MACS2 in broad mode using the matched input control and hg19 coordinates. Merge the called broad peaks within each dataset with bedtools merge to obtain a non-redundant peak catalog. Compute the merged-peak length distribution and derive a dataset-specific size cutoff with findiplist() from the R inflection package, using the upper limit value of the inflection-point analysis (expected ~6689 bp for GSE24447, ~6719 bp for GSE108518, ~8020 bp for GSE85628). Overlap merged peaks exceeding the dataset-specific cutoff with hg19 gene bodies and binarize each gene to 0/1. The rationale is that broad H3K27me3 domains mark repressed polycomb-associated promoters, so only exceptionally broad peaks indicate polycomb-silenced genes.

## Related tools

- **MACS2** (Calls broad H3K27me3 peaks in broad mode using matched input controls)
- **bedtools** (Merges overlapping broad peaks within each dataset into a non-redundant peak catalog and overlaps peaks with hg19 gene bodies)
- **findiplist (R inflection package)** (Inflection-point analysis of the merged-peak length distribution to derive the per-dataset peak-size cutoff)
- **R** (Runtime environment (>=3.5.0) for the cutoff computation and polycombScore binarization) — https://github.com/vicsanga/Postre
- **POSTRE** (Consumes the resulting polycombScore.tsv as part of its cell type-specific regulatory annotations) — https://github.com/vicsanga/Postre

## Examples

```
macs2 callpeak -t H3K27me3.bam -c input.bam -f BAMPE -g hs --broad --broad-cutoff 0.1 -n H3K27me3_broad
```

## Evaluation signals

- findiplist() cutoffs fall near the expected values: ~6689 bp for GSE24447, ~6719 bp for GSE108518, ~8020 bp for GSE85628
- Every gene row in polycombScore.tsv contains a binary 0/1 score
- Merged peak catalog is non-redundant after bedtools merge within each dataset
- Output file is named polycombScore.tsv and is a tab-separated plain text table
- Peaks are called in hg19 coordinates, matching POSTRE's coordinate system for SV breakpoints

## Limitations

- Cutoffs are dataset-specific: the inflection-point size threshold must be re-derived per cell type/dataset and does not transfer between them
- Quality depends on the availability of matched input controls for MACS2 peak calling
- POSTRE's overall predictions depend on incomplete characterization of enhancers and regulatory data across human cell types, so derived modules may miss relevant biology for some tissues

## Evidence

- [supplementary] POSTRE derives a per-cell-type H3K27me3 peak-size cutoff from the upper limit value of the findiplist() inflection-point analysis: [evidence span withheld — anchor status: not_found]
- [supplementary] Call broad H3K27me3 peaks with MACS2 in broad mode, using the matched input control for each dataset and hg19 coordinates: [evidence span withheld — anchor status: not_found]
- [supplementary] Merge the called broad peaks within each dataset using bedtools merge to obtain a non-redundant peak catalog.: [evidence span withheld — anchor status: not_found]
