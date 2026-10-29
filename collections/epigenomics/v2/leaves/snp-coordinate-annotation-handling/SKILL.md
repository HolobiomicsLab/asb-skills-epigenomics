---
name: snp-coordinate-annotation-handling
description: Use when tabular coordinate data (SNP positions, peak intervals) must
  be converted into GRanges objects with metadata columns as preprocessing before
  range-based enrichment or bootstrapped null-set analyses.
when_to_use_negative:
- Input ranges are already GRanges objects with metadata, ready for segBootranges.R
- The downstream analysis requires only simple overlaps and no per-range annotations
  such as SNP identifiers or peak statistics
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: bootRanges
  role: Downstream consumer of the constructed GRanges objects; segBootranges.R performs
    bootRanges on the saved RDS inputs
  repo: https://bioconductor.org/packages/nullranges
- name: nullranges
  role: R/Bioconductor package hosting bootRanges used after GRanges construction
  repo: https://bioconductor.org/packages/nullranges
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad190
    title: 'bootRanges: flexible generation of null sets of genomic ranges for hypothesis
      testing'
  source_content_hash: sha256:899f904e9f2c7fdb06ad81e10947f7ed5193c906675d13ebebb09b4eae432906
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - bootRanges
  - nullranges
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# snp-coordinate-annotation-handling

## Summary

Constructs GRanges objects from tabular SNP/peak coordinate tables (e.g. total-cholesterol-associated SNPs and liver ATAC-seq caQTL peaks) so that downstream null-model tools like bootRanges can operate on them; attaches annotations as mcols metadata for computing test statistics beyond simple overlaps.

## When to use

Use when tabular coordinate data (SNP positions, peak intervals) must be converted into GRanges objects with metadata columns as preprocessing before range-based enrichment or bootstrapped null-set analyses.

## When NOT to use

- Input ranges are already GRanges objects with metadata, ready for segBootranges.R
- The downstream analysis requires only simple overlaps and no per-range annotations such as SNP identifiers or peak statistics

## Inputs

- liver ATAC-seq peak (caQTL) coordinate table loaded in R
- total-cholesterol-associated SNP table loaded in R

## Outputs

- peak GRanges object saved as RDS
- SNP GRanges object saved as RDS with metadata columns in mcols

## How to apply

Load the coordinate tables (e.g. the liver ATAC-seq caQTL peak table and the total-cholesterol-associated SNP table) into R and convert each into a GRanges object with GenomicRanges, mapping seqnames, start/end ranges, and strand from tabular columns. Attach remaining columns such as SNP identifiers and peak statistics as metadata via mcols, since the workflow (GRanges_construct.R) notes optional metadata columns are usable for computing test statistics beyond simple overlaps. Confirm both GRanges use a consistent reference genome build so subsequent overlap operations are valid. Save the peak GRanges and SNP GRanges as RDS files to serve as inputs for the bootRanges step (segBootranges.R) in Application 3.1.

## Related tools

- **bootRanges** (Downstream consumer of the constructed GRanges objects; segBootranges.R performs bootRanges on the saved RDS inputs) — https://bioconductor.org/packages/nullranges
- **nullranges** (R/Bioconductor package hosting bootRanges used after GRanges construction) — https://bioconductor.org/packages/nullranges

## Examples

```
gr_peaks <- GRanges(seqnames, IRanges(start, end), strand); mcols(gr_peaks) <- peak_stats; saveRDS(gr_peaks, "peaks.rds")
```

## Evaluation signals

- Saved RDS files load back as GRanges objects with the expected number of ranges
- mcols contain non-empty metadata columns (field_present check)
- Both GRanges use a consistent reference genome build so overlap operations are valid
- seqnames, start/end, and strand correctly mapped from the source tabular columns

## Limitations

- Coordinate tables must come from a consistent reference genome build or subsequent overlap operations will be invalid
- This step only preprocesses inputs; it does not itself generate null sets or compute statistics

## Evidence

- [other] The preprocessing step (GRanges_construct.R) constructs the workflow input as range sets x and y — here the liver ATAC-seq peaks and the cholesterol SNPs — stored as GRanges objects: [evidence span withheld — anchor status: not_found]
- [other] Convert each coordinate table into a GRanges object using GenomicRanges, mapping seqnames, start/end ranges, and strand from the tabular columns: [evidence span withheld — anchor status: not_found]
- [readme] GRanges_construct.R (Preprocess) -> segBootranges.R (perform bootRanges): "*GRanges_construct.R*(Preprocess) -> *segBootranges.R*(perform bootRanges) -> *var.R*(Variance of the rate of overlaps and $z$ score)"
- [intro] Derive overlap rate of caQTLs detected in human liver tissue-SNPs associated with total cholesterol: "Derive overlap rate of  caQTLs detected in human liver tissue-SNPs associated with total cholesterol"
