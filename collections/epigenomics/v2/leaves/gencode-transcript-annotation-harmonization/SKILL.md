---
name: gencode-transcript-annotation-harmonization
description: Use when harmonizing lncRNA identifiers and sequences across multiple
  published RNA:DNA interaction datasets (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq)
  prior to peak calling or benchmarking, so all entries share one gene-symbol and
  transcript reference.
when_to_use_negative:
- The input is a single self-consistent dataset already annotated in its native reference
  (e.g. a published ChIRP-seq peak file with its own gene symbols) and no cross-dataset
  comparison is planned
- Protein-coding or small-RNA annotations are the target and transcript isoform resolution
  beyond longest-transcript selection is required — this skill collapses each gene
  to one representative transcript
edam_operation: http://edamontology.org/operation_0361
tools:
- name: GENCODE version 32
  role: Reference annotation source providing gene symbols (v32 human, M25 mouse)
    and transcript sequences used to uniformly rename and represent every RNA molecule
    across datasets
- name: GEO
  role: Source database whose records supply the RNA names that must be mapped to
    GENCODE symbols
- name: ENA
  role: Source database whose records supply the RNA names that must be mapped to
    GENCODE symbols
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.016
    title: 3plex enables deep computational investigation of triplex forming lncRNAs
  source_content_hash: sha256:0b6119886ab08ff15155265590e2e12c5d4461abdd9d10b71e60cdcc9c33acee
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/molinerisLab/3plex
  tools:
  - 3plex
  - ENA
  - GENCODE version 32
  - GEO
  edam_operation: http://edamontology.org/operation_0361
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gencode-transcript-annotation-harmonization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uniform re-annotation of RNA molecules from heterogeneous experimental lncRNA:DNA binding datasets using GENCODE version 32 (human, v32) and M25 (mouse) gene symbols, with a single canonical sequence per gene, so that datasets collected from GEO, PubMed and ENA become directly comparable.

## When to use

Use when harmonizing lncRNA identifiers and sequences across multiple published RNA:DNA interaction datasets (ChIRP-seq, ChOP-seq, CHART-seq, RAP-seq) prior to peak calling or benchmarking, so all entries share one gene-symbol and transcript reference.

## When NOT to use

- The input is a single self-consistent dataset already annotated in its native reference (e.g. a published ChIRP-seq peak file with its own gene symbols) and no cross-dataset comparison is planned
- Protein-coding or small-RNA annotations are the target and transcript isoform resolution beyond longest-transcript selection is required — this skill collapses each gene to one representative transcript

## Inputs

- List of lncRNAs and RNA identifiers harvested from GEO, PubMed and ENA records of lncRNA:DNA interaction experiments (2011-2022)
- GENCODE version 32 (human) and GENCODE M25 (mouse) annotation files
- Original publication sequences for RNAs lacking a GENCODE entry (e.g. lncSmad7)

## Outputs

- Harmonized per-lncRNA transcript sequences (longest transcript per gene) in FASTA form for downstream TPX prediction
- Uniform gene-symbol mapping applied to the final binding_site_peaks.bed benchmark peak sets

## How to apply

After downloading raw fastq files for the collected lncRNA:DNA interaction experiments and re-analysing them with the ENCODE ChIP-seq pipeline (v1.6.1), map every RNA mentioned in each dataset to its official gene symbol using GENCODE version 32 for human and M25 for mouse. For each gene, take the longest transcript as the representative sequence to avoid fragment-level ambiguity across datasets. Handle known exceptions explicitly — e.g. lncSmad7, whose sequence was taken from its original publication rather than GENCODE. This uniform annotation is what allows the final per-lncRNA peak sets (IDR conservative, IDR optimal, Overlap conservative q<0.05, Overlap optimal, Top1000) to be written to binding_site_peaks.bed and used as a consistent benchmark for 3plex evaluation.

## Related tools

- **GENCODE version 32** (Reference annotation source providing gene symbols (v32 human, M25 mouse) and transcript sequences used to uniformly rename and represent every RNA molecule across datasets)
- **GEO** (Source database whose records supply the RNA names that must be mapped to GENCODE symbols)
- **ENA** (Source database whose records supply the RNA names that must be mapped to GENCODE symbols)

## Evaluation signals

- Every RNA molecule in the processed datasets resolves to exactly one GENCODE v32/M25 gene symbol with no unmapped identifiers
- Each gene is represented by exactly one sequence — the longest transcript per gene — with lncSmad7 explicitly replaced by its original-publication sequence
- The output binding_site_peaks.bed exists, parses as valid BED, and contains genomic intervals only for lncRNAs whose datasets yielded peak sets under the applicable filtering methods
- Cross-dataset checks show no duplicate gene symbols with conflicting sequences in the harmonized transcript set

## Limitations

- Longest-transcript selection discards isoform information, which may matter if a specific isoform mediates the triplex interaction
- RNAs absent from GENCODE v32/M25 require manual retrieval from the original publication (as done for lncSmad7); datasets lacking such sequences cannot be harmonized automatically
- Only 2 of the 7 TPX-validated lncRNAs had IDR conservative peak sets, so the harmonized benchmark is uneven in reproducibility quality across lncRNAs

## Evidence

- [methods] RNA molecules were uniformly annotated using GENCODE version 32 and M25 gene symbols [36]: "RNA molecules were uniformly annotated using GENCODE version 32 and M25 gene symbols [36]"
- [other] Annotate all RNA molecules uniformly with GENCODE version 32 and M25 gene symbols, taking the longest transcript per gene (lncSmad7 sequence taken from its original publication), and write the final peak sets to binding_site_peaks.bed.: [evidence span withheld — anchor status: not_found]
- [other] The final peak sets were written to binding_site_peaks.bed with valid BED-formatted genomic intervals for the processed lncRNAs.: [evidence span withheld — anchor status: not_found]
