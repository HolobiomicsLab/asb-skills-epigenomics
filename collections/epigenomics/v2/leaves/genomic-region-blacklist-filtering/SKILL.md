---
name: genomic-region-blacklist-filtering
description: Reference for removing non-informative or artifact-prone regions (sex
  chromosomes, chrM, ENCODE blacklist) from bin-sample methylation quantification
  matrices before downstream group/batch comparisons.
when_to_use_negative:
- Input is a per-sample (not aggregated) methylation quantification file where filtering
  should be handled by the upstream quantification tool
- The analysis intentionally requires sex chromosome or mitochondrial methylation
  signal (e.g. studying sex-chromosome methylation), since this filter uniformly removes
  those bins
- Quantification matrices are already filtered or the downstream method expects blacklist
  regions to be handled with its own annotation scheme
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Snakemake
  role: Orchestrates the aggregation, filtering, sort/bgzip, and Tabix-indexing steps
    of the MEDIPIPE module
  repo: https://github.com/pughlab/MEDIPIPE
- name: Tabix
  role: Indexes the sorted, bgzip-compressed original and filtered bin-sample matrices
    for rapid genomic region retrieval (v1.17)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad423
    title: 'MEDIPIPE: an automated and comprehensive pipeline for cfMeDIP-seq data
      quality control and analysis'
  source_content_hash: sha256:30fb50dd2fa4b0d324c5508e892f4f2d22fc1b39e83a6ac235369ebc4ef03217
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/pughlab/MEDIPIPE
  tools:
  - MultiQC
  - Snakemake
  - Tabix
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genomic-region-blacklist-filtering

## Summary

Uniformly filters aggregated bin-sample methylation matrices produced by cfMeDIP-seq pipelines (MEDIPIPE-style) by removing genomic bins on sex chromosomes, the mitochondrial chromosome, and ENCODE blacklist regions (Amemiya et al. 2019), producing both original and filtered TAB-delimited outputs that are bgzip-compressed and Tabix-indexed.

## When to use

Reference for removing non-informative or artifact-prone regions (sex chromosomes, chrM, ENCODE blacklist) from bin-sample methylation quantification matrices before downstream group/batch comparisons.

## When NOT to use

- Input is a per-sample (not aggregated) methylation quantification file where filtering should be handled by the upstream quantification tool
- The analysis intentionally requires sex chromosome or mitochondrial methylation signal (e.g. studying sex-chromosome methylation), since this filter uniformly removes those bins
- Quantification matrices are already filtered or the downstream method expects blacklist regions to be handled with its own annotation scheme

## Inputs

- Aggregated TAB-delimited bin-sample TXT methylation quantification matrices (rows = genomic bins, columns = samples) from MEDIPS, QSEA, or MEDStrand
- ENCODE blacklist regions file (Amemiya et al. 2019)
- Chromosome coordinate annotations for genomic bins

## Outputs

- Original (unfiltered) sorted, bgzip-compressed, Tabix-indexed bin-sample matrix
- Filtered bin-sample matrix with sex chromosome, chrM, and blacklist-region bins removed; sorted, bgzip-compressed, and Tabix-indexed
- HTML QC report (via MultiQC) contextualizing the aggregated samples

## How to apply

After aggregating per-sample methylation quantifications (from MEDIPS, QSEA, or MEDStrand) into TAB-delimited bin-sample TXT matrices (rows = genomic bins, columns = samples), apply the filter uniformly across all quantification types. Remove all bins mapping to sex chromosomes, the mitochondrial chromosome, and regions listed in the ENCODE blacklist (Amemiya et al. 2019), so that aggregation and filtering decisions do not depend on sample group or batch. Keep both the original and the filtered matrix versions so users can trace what was removed. Finally, sort and bgzip-compress both matrices and index them with Tabix (v1.17) so genomic regions of interest can be rapidly retrieved by region query.

## Related tools

- **Snakemake** (Orchestrates the aggregation, filtering, sort/bgzip, and Tabix-indexing steps of the MEDIPIPE module) — https://github.com/pughlab/MEDIPIPE
- **Tabix** (Indexes the sorted, bgzip-compressed original and filtered bin-sample matrices for rapid genomic region retrieval (v1.17))

## Examples

```
bgzip filtered_matrix.txt && tabix -s 1 -b 2 -e 2 filtered_matrix.txt.gz
```

## Evaluation signals

- Both the original and the filtered aggregated quantification files exist for every quantification type (MEDIPS, QSEA, MEDStrand)
- No bins from sex chromosomes, the mitochondrial chromosome, or ENCODE blacklist regions (Amemiya et al. 2019) remain in the filtered matrix
- Both original and filtered matrices are sorted, bgzip-compressed, and Tabix-indexed
- A Tabix region query against the filtered matrix returns only bins within the queried interval
- The HTML QC report renders selected QC metrics for all aggregated samples (e.g. the 163-sample demonstration set spanning 6 brain cancer subtypes)

## Limitations

- The filter uniformly removes all sex chromosome, chrM, and blacklist bins — analyses targeting these regions cannot use the filtered output; the original matrix must be retained and used instead
- Blacklist definition is fixed to ENCODE blacklist regions (Amemiya et al. 2019); region sets from other projects or genome builds require reconfiguration
- Filtering requires prior successful aggregation into bin-sample matrices; it cannot recover signal lost by upstream quantification steps

## Evidence

- [results] MEDIPIPE's aggregation module combines per-sample methylation quantifications into TAB-delimited bin-sample TXT matrices and uniformly filters them: [evidence span withheld — anchor status: not_found]
- [results] ENCODE blacklist regions from Amemiya et al. 2019 are the filter's region set: [evidence span withheld — anchor status: not_found]
- [results] Tabix indexing enables rapid retrieval of genomic regions of interest: [evidence span withheld — anchor status: not_found]
- [readme] README description of the full MEDIPIPE scope including aggregation: "The pipeline starts from the raw FASTQ files all the way to QC, alignment, methylation quantifications and aggregation."
- [readme] README statement of applicability to large-scale sample aggregation: "It can be applied to individual samples, as well as to aggregate multiple samples from large-scale profiling."
