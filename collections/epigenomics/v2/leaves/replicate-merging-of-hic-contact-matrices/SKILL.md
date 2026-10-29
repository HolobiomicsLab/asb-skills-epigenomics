---
name: replicate-merging-of-hic-contact-matrices
description: Use when multiple Hi-C replicates (e.g., QSC and FISC MuSC replicates)
  exist as Microcket-produced .hic files and a per-condition contact matrix/heatmap
  is needed for specific regions and bin sizes.
when_to_use_negative:
- Input is raw FASTQ or unaligned pairs — this skill operates on Microcket-generated
  .hic files produced by the upstream Microcket run
- Only one replicate per condition exists and no merging is required — extracting
  and plotting a single matrix is sufficient
- The goal is calling TADs/loops or other quantitative analysis rather than visual
  contact-map rendering for expert review
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Microcket
  role: Upstream tool that generates the .hic contact matrix files for each QSC and
    FISC replicate that are merged in this skill
  repo: https://github.com/hellosunking/Microcket
- name: JuicerTools
  role: Used with straw/dump to extract KR-normalized contact matrices from the .hic
    files at the requested chr16 regions and bin sizes; Juicebox is the underlying
    Hi-C visualization software
  repo: https://github.com/aidenlab/JuicerTools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2025.103962
    title: Protocol for investigating chromatin architecture in adult skeletal muscle
      stem cells using high-throughput chromosome conformation capture
  source_content_hash: sha256:fe6e8a19fbde91f3412e75898df2dd215339e1a2e41a4c0c3b4ad6b1bc7eef13
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# Replicate merging of Hi-C contact matrices

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges per-replicate Microcket-generated .hic contact matrices into a single KR-normalized contact matrix per condition (QSC, FISC) and renders them as side-by-side chr16 contact-map heatmaps at 100-kb, 25-kb, and 5-kb resolutions for comparative chromatin architecture analysis.

## When to use

Use when multiple Hi-C replicates (e.g., QSC and FISC MuSC replicates) exist as Microcket-produced .hic files and a per-condition contact matrix/heatmap is needed for specific regions and bin sizes.

## When NOT to use

- Input is raw FASTQ or unaligned pairs — this skill operates on Microcket-generated .hic files produced by the upstream Microcket run
- Only one replicate per condition exists and no merging is required — extracting and plotting a single matrix is sufficient
- The goal is calling TADs/loops or other quantitative analysis rather than visual contact-map rendering for expert review

## Inputs

- Microcket-generated .hic contact matrix files for all QSC replicates
- Microcket-generated .hic contact matrix files for all FISC replicates
- Region/bin-size specifications: chr16 50 Mb at 100,000 bp, 10 Mb at 25,000 bp, 1.5 Mb at 5,000 bp

## Outputs

- Merged KR-normalized contact matrix per condition (QSC, FISC) at each of the three resolutions
- Square contact-map heatmap panels per region/resolution with QSC and FISC side by side
- qsc_fisc_chr16_contact_maps.png containing all rendered contact maps

## How to apply

Collect the Microcket-generated .hic contact matrix files for all QSC replicates and all FISC replicates. Use JuicerTools (straw/dump) to extract Knight-Ruiz (KR)-normalized contact matrices for each requested view: a 50 Mb region of chr16 binned at 100,000 bp, a 10 Mb region at 25,000 bp, and a 1.5 Mb region at 5,000 bp. Merge the per-replicate matrices into one contact matrix per condition (QSC, FISC) at each resolution, then render each merged matrix as a square heatmap with QSC and FISC arranged side by side so cis-interaction patterns are directly comparable. Export all panels into a single image file (qsc_fisc_chr16_contact_maps.png). Rationale: merging replicates before rendering ensures the visual comparison reflects the condition-level architecture rather than replicate-specific noise, and matching region/bin-size pairs between conditions is what makes the side-by-side contrast interpretable.

## Related tools

- **Microcket** (Upstream tool that generates the .hic contact matrix files for each QSC and FISC replicate that are merged in this skill) — https://github.com/hellosunking/Microcket
- **JuicerTools** (Used with straw/dump to extract KR-normalized contact matrices from the .hic files at the requested chr16 regions and bin sizes; Juicebox is the underlying Hi-C visualization software) — https://github.com/aidenlab/JuicerTools

## Examples

```
straw KR QSC.hic chr16:0:50000000 chr16:0:50000000 BP 100000 > qsc_chr16_50Mb_100kb.txt
```

## Evaluation signals

- qsc_fisc_chr16_contact_maps.png exists and contains all rendered contact-map panels
- Each exported panel spans exactly the requested chr16 region at the requested bin size (50 Mb @ 100-kb, 10 Mb @ 25-kb, 1.5 Mb @ 5-kb)
- Both QSC and FISC contact maps are present and side by side for every region/resolution combination
- Each merged matrix is a single per-condition matrix derived from all replicates of that condition, not a single replicate
- Visual QC: contrast of cis-interaction patterns between QSC and FISC is assessed by expert review

## Limitations

- Final visual QC of QSC vs FISC cis-interaction contrast is assessed by expert review and is therefore subjective
- KR-normalized extraction via JuicerTools requires that normalization/balancing was computed in the .hic files produced by Microcket
- The Microcket protocol itself has only been evaluated on MuSCs from Pax7-nGFP and C57BL/6 mice, C2C12 myoblasts, and HeLa-S3 and K562 human cell lines

## Evidence

- [results] Finding: Hi-C interaction maps from QSC/FISC replicates: [evidence span withheld — anchor status: not_found]
- [readme] Juicebox as Hi-C visualization software: [evidence span withheld — anchor status: external_doc]
