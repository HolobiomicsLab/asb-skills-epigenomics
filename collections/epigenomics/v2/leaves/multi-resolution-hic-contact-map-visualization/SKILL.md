---
name: multi-resolution-hic-contact-map-visualization
description: Use when hic contact matrix files from the Microcket pipeline are available
  and region-specific contact-map heatmaps must be extracted and rendered at multiple
  resolutions for side-by-side condition comparison.
when_to_use_negative:
- Input is raw Hi-C FASTQ or unaligned pairs — the .hic files must already exist from
  the upstream Microcket run; this skill does no alignment or filtering (e.g., no
  minimum mapping quality filter at default 10).
- The analysis requires genome-wide quantitative scoring (e.g., compartment eigenvectors
  or loop calling) rather than visual region-level contact maps.
- Data are stored in .cool/.cooler format without a .hic conversion path — straw/dump
  operate on .hic.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Microcket
  role: Upstream pipeline that produces the .hic contact matrix files consumed by
    this visualization skill
  repo: https://github.com/hellosunking/Microcket
- name: JuicerTools
  role: straw/dump extraction of KR-normalized contact matrices from .hic files at
    the requested region and bin size
  repo: https://github.com/aidenlab/JuicerTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2025.103962
    title: Protocol for investigating chromatin architecture in adult skeletal muscle
      stem cells using high-throughput chromosome conformation capture
  source_content_hash: sha256:51d7cd27a0207aea11ddf2de92f0e53ae151dd48ecf63f8c1409ccf02e50dc2a
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# multi-resolution-hic-contact-map-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts KR-normalized contact matrices from Microcket-generated .hic files with JuicerTools (straw/dump) and renders square cis-contact heatmaps at multiple bin sizes (e.g., 100-kb, 25-kb, 5-kb) so that chromatin architecture differences between conditions such as QSC and FISC are directly comparable.

## When to use

Use when .hic contact matrix files from the Microcket pipeline are available and region-specific contact-map heatmaps must be extracted and rendered at multiple resolutions for side-by-side condition comparison.

## When NOT to use

- Input is raw Hi-C FASTQ or unaligned pairs — the .hic files must already exist from the upstream Microcket run; this skill does no alignment or filtering (e.g., no minimum mapping quality filter at default 10).
- The analysis requires genome-wide quantitative scoring (e.g., compartment eigenvectors or loop calling) rather than visual region-level contact maps.
- Data are stored in .cool/.cooler format without a .hic conversion path — straw/dump operate on .hic.

## Inputs

- Microcket-generated .hic contact matrix files for all QSC replicates
- Microcket-generated .hic contact matrix files for all FISC replicates
- Region and binning specifications (e.g., chr16: 50 Mb @ 100 kb, 10 Mb @ 25 kb, 1.5 Mb @ 5 kb)
- JuicerTools jar (straw/dump)

## Outputs

- KR-normalized per-replicate contact matrices per region/resolution
- Merged per-condition (QSC, FISC) contact matrices at each resolution
- Rendered contact-map heatmap panels (QSC and FISC side by side) exported to qsc_fisc_chr16_contact_maps.png

## How to apply

Collect the Microcket-generated .hic files for all replicates of each condition (QSC and FISC in the source study). Use JuicerTools straw/dump to extract KR-normalized cis contact matrices for each requested view — here a 50 Mb region of chr16 at 100,000 bp bins, a 10 Mb region at 25,000 bp bins, and a 1.5 Mb region at 5,000 bp bins. Merge per-replicate matrices into one contact matrix per condition at each resolution so the heatmap reflects the full replicate set. Render each merged matrix as a square heatmap, arranging QSC and FISC side by side for the same region/resolution so cis-interaction patterns (TADs, loops, compartments at the visible scale) can be visually contrasted. Export all panels into a single image, qsc_fisc_chr16_contact_maps.png. The rationale is that coarse bins reveal compartment-scale architecture while fine bins resolve sub-Mb structures, and Juicebox/JuicerTools is the visualization software designed for .hic data.

## Related tools

- **Microcket** (Upstream pipeline that produces the .hic contact matrix files consumed by this visualization skill) — https://github.com/hellosunking/Microcket
- **JuicerTools** (straw/dump extraction of KR-normalized contact matrices from .hic files at the requested region and bin size) — https://github.com/aidenlab/JuicerTools

## Examples

```
java -jar juicer_tools.jar straw KR QSC_merged.hic chr16:1:50000000 BP 100000 > qsc_chr16_100kb.txt
```

## Evaluation signals

- qsc_fisc_chr16_contact_maps.png exists and is openable as an image.
- Each exported panel spans exactly the requested chr16 region at the requested bin size (50 Mb @ 100 kb, 10 Mb @ 25 kb, 1.5 Mb @ 5 kb).
- Both QSC and FISC maps are present side by side for each region/resolution combination.
- Extracted matrix dimensions match the region/bin-size ratio (e.g., 50x50 bins for the 50 Mb @ 100 kb view).
- Visual QC of cis-interaction pattern contrast between QSC and FISC is assessed by expert review.

## Limitations

- Visual QC is qualitative and relies on expert review rather than an automated metric.
- MuSCs Hi-C datasets are pre-existing public data (GSE189841); the protocol did not generate new datasets, so region choices are fixed to the published views unless new .hic files are supplied.
- Microcket is distributed under GPLv3 for personal and academic usage only.

## Evidence

- [results] Multi-resolution chr16 contact maps: [evidence span withheld — anchor status: not_found]
- [abstract] Hi-C protocol scope: "Here, we present a protocol for investigating chromatin architecture in adult skeletal muscle stem cells, also known as satellite cells (MuSCs), using Hi-C."
- [discussion] Source datasets: "The MuSCs Hi-C datasets analyzed in this study are available at Gene Expression Omnibus database: GSE189841."
- [readme] Juicebox role for .hic visualization: [evidence span withheld — anchor status: external_doc]
