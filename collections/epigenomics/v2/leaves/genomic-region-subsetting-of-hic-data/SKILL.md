---
name: genomic-region-subsetting-of-hic-data
description: Use when Microcket-generated .hic contact-matrix files (e.g., QSC and
  FISC Hi-C replicates) must be visualized as contact maps for specific regions, resolutions,
  and downstream comparative analyses.
when_to_use_negative:
- The raw data are still FASTQ reads and have not yet been processed by Microcket
  into .hic files — run the upstream Microcket pipeline first
- The analysis goal is genome-wide quantification (e.g., compartment or TAD calling)
  rather than subsetting and visualizing specific genomic regions
- The .hic files were not balanced/normalized (KR normalization unavailable), making
  cross-condition comparison of contact intensities unreliable
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0121
tools:
- name: Microcket
  role: Upstream pipeline that produces the .hic contact matrix files for each Hi-C
    replicate that this skill subsets and visualizes
  repo: https://github.com/hellosunking/Microcket
- name: JuicerTools
  role: Provides straw/dump commands to extract KR-normalized contact matrices from
    .hic files at specified regions and bin sizes
  repo: https://github.com/aidenlab/JuicerTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2025.103962
    title: Protocol for investigating chromatin architecture in adult skeletal muscle
      stem cells using high-throughput chromosome conformation capture
  source_content_hash: sha256:82f62fd946e078c1773a18333d442ca495bd1844c4a5ed48341b019c955206fa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/hellosunking/Microcket
  repo_url: https://github.com/hellosunking/Microcket
  tools:
  - JuicerTools
  - Microcket
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# genomic-region-subsetting-of-hic-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract KR-normalized Hi-C contact matrices for defined chromosomal regions and bin sizes from .hic files using JuicerTools (straw/dump), merge per-replicate matrices per condition, and render square contact-map heatmaps (e.g., chr16 views of QSC vs FISC) for direct visual comparison of chromatin architecture.

## When to use

Use when Microcket-generated .hic contact-matrix files (e.g., QSC and FISC Hi-C replicates) must be visualized as contact maps for specific regions, resolutions, and downstream comparative analyses.

## When NOT to use

- The raw data are still FASTQ reads and have not yet been processed by Microcket into .hic files — run the upstream Microcket pipeline first
- The analysis goal is genome-wide quantification (e.g., compartment or TAD calling) rather than subsetting and visualizing specific genomic regions
- The .hic files were not balanced/normalized (KR normalization unavailable), making cross-condition comparison of contact intensities unreliable

## Inputs

- Microcket-generated .hic contact matrix files for all QSC replicates and all FISC replicates
- Region/resolution specifications (e.g., chr16: 50 Mb at 100-kb, 10 Mb at 25-kb, 1.5 Mb at 5-kb bins)
- KR normalization choice for JuicerTools straw/dump extraction

## Outputs

- KR-normalized contact matrices extracted per replicate per region/resolution
- Merged per-condition contact matrices (QSC, FISC) at each resolution
- Rendered contact-map heatmap panels combined into qsc_fisc_chr16_contact_maps.png

## How to apply

Collect the Microcket-generated .hic files for all replicates of each condition (e.g., all QSC replicates and all FISC replicates from the upstream Microcket run). Use JuicerTools straw/dump to extract KR-normalized contact matrices for each requested view — here a 50 Mb region of chr16 binned at 100,000 bp, a 10 Mb region at 25,000 bp, and a 1.5 Mb region at 5,000 bp — invoking straw with the KR normalization and the BP resolution (e.g., BP@100000). Merge the extracted per-replicate matrices into a single contact matrix per condition at each resolution so replicates are pooled before visualization. Render each merged matrix as a square contact-map heatmap, arranging QSC and FISC side by side for the same region/resolution so cis-interaction differences are directly comparable, and export all panels into a single image (e.g., qsc_fisc_chr16_contact_maps.png). Validation relies on confirming the exported image exists and that each panel spans exactly the requested chr16 region at the requested bin size with both conditions present; contrast of cis-interaction patterns is assessed by expert visual QC.

## Related tools

- **Microcket** (Upstream pipeline that produces the .hic contact matrix files for each Hi-C replicate that this skill subsets and visualizes) — https://github.com/hellosunking/Microcket
- **JuicerTools** (Provides straw/dump commands to extract KR-normalized contact matrices from .hic files at specified regions and bin sizes) — https://github.com/aidenlab/JuicerTools

## Examples

```
straw KR qsc.hic chr16:0:50000000 chr16:0:50000000 BP@100000 qsc_chr16_100kb_KR.txt
```

## Evaluation signals

- qsc_fisc_chr16_contact_maps.png exists and opens as a valid image
- Each exported panel spans exactly the requested chr16 region (50 Mb / 10 Mb / 1.5 Mb) at the requested bin size (100 kb / 25 kb / 5 kb)
- Both QSC and FISC contact maps are present for every region/resolution combination
- Extracted matrices are KR-normalized and per-replicate matrices merge cleanly into one matrix per condition without dimension mismatches
- Expert visual QC confirms interpretable cis-interaction patterns with contrast sufficient to compare QSC vs FISC chromatin architecture

## Limitations

- Visual QC of cis-interaction contrast is assessed by expert review rather than an automated metric
- The Microcket pipeline that generates the .hic inputs depends on external tools (Ktrim, FLASH, BWA, Samtools, JuicerTools) and hg38/mm10 reference packages; missing dependencies or low valid-pair rates upstream propagate into the contact maps

## Evidence

- [abstract] Microcket and Hi-C visualization workflow: "introduce the use of the Microcket package for processing Hi-C data"
- [methods] JuicerTools as part of the Microcket toolchain: "- [JuicerTools](https://github.com/aidenlab/JuicerTools "JuicerTools")"
- [readme] JuicerTools/Juicebox purpose: [evidence span withheld — anchor status: external_doc]
- [discussion] Source Hi-C datasets for QSC and FISC: "The MuSCs Hi-C datasets analyzed in this study are available at Gene Expression Omnibus database: GSE189841."
