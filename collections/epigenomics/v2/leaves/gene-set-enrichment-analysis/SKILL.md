---
name: gene-set-enrichment-analysis
description: Use when genes must be ranked by epigenomic signal (e.g. RNAP2 or H3K4me3
  ChIP-seq prevalence) within chromosome clusters such as BHi-Cect bottom enclaves,
  and tested for GO-term enrichment via GSEA with Benjamini-Hochberg FDR correction.
when_to_use_negative:
- The analysis goal is contiguous TAD-level segmentation of Hi-C data rather than
  non-contiguous, deeply interwoven enclave clusters — BHi-Cect explicitly clusters
  without imposing genomic contiguity
- ChIP-seq signal has not been aggregated per enclave; GSEA requires per-gene (or
  per-enclave) ranked lists, not raw contact matrices
- The cell lines or marks differ from RNAP2/IMR90 and H3K4me3/K562 without re-validating
  the expected enrichment categories
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0622
tools:
- name: BHi-Cect
  role: Top-down spectral clustering of Hi-C data producing the nested tree and bottom
    enclave locus assignments used to rank genes for GSEA
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Environment in which BHi-Cect functions are copy-pasted and the GSEA aggregation/ranking
    of ChIP-seq signal per enclave is performed
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Used to aggregate mean ChIP-seq signal per bottom enclave and prepare ranked
    gene lists
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:09ab075555e24549274572f63a588cc7d9660d767355eab06ff81b3edd3a106b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# gene-set-enrichment-analysis

## Summary

Runs GSEA with Gene Ontology gene sets on gene lists ranked by RNAP2 (IMR90) or H3K4me3 (K562) ChIP-seq signal prevalence in bottom enclaves identified by the BHi-Cect hierarchical clustering of Hi-C data, linking chromosome enclave structure to enriched biological functions.

## When to use

Use when genes must be ranked by epigenomic signal (e.g. RNAP2 or H3K4me3 ChIP-seq prevalence) within chromosome clusters such as BHi-Cect bottom enclaves, and tested for GO-term enrichment via GSEA with Benjamini-Hochberg FDR correction.

## When NOT to use

- The analysis goal is contiguous TAD-level segmentation of Hi-C data rather than non-contiguous, deeply interwoven enclave clusters — BHi-Cect explicitly clusters without imposing genomic contiguity
- ChIP-seq signal has not been aggregated per enclave; GSEA requires per-gene (or per-enclave) ranked lists, not raw contact matrices
- The cell lines or marks differ from RNAP2/IMR90 and H3K4me3/K562 without re-validating the expected enrichment categories

## Inputs

- RNAP2 ChIP-seq signal data for IMR90
- H3K4me3 ChIP-seq signal data for K562
- Bottom enclave locus assignments from BHi-Cect tree output (nested cluster list of loci per cluster)
- Gene genomic locus coordinates (for mapping genes to enclaves)
- Gene Ontology (GO) gene sets

## Outputs

- gsea_results.csv containing FDR-corrected enrichment scores and gene-set hits per bottom enclave
- Per-enclave ranked gene lists (by RNAP2 or H3K4me3 prevalence)

## How to apply

Load the ChIP-seq signal data (RNAP2 for IMR90, H3K4me3 for K562) and the bottom enclave locus assignments from the BHi-Cect tree output (nested cluster list). Map each gene's genomic locus to its bottom enclave and aggregate mean ChIP-seq signal per enclave using dplyr in R, then rank genes by signal intensity within each bottom enclave separately per cell line and histone/TF mark. Run GSEA with GO gene sets on each per-enclave ranked list and apply Benjamini-Hochberg FDR correction across all tested gene sets. Judge success by comparing reproduced enrichment categories against the reported results: skin function, structural molecule activity (cytoskeleton) and DNA packaging for IMR90 RNAP2; immune functions for K562 H3K4me3. Write the FDR-corrected enrichment scores and gene-set hits to gsea_results.csv.

## Related tools

- **BHi-Cect** (Top-down spectral clustering of Hi-C data producing the nested tree and bottom enclave locus assignments used to rank genes for GSEA) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Environment in which BHi-Cect functions are copy-pasted and the GSEA aggregation/ranking of ChIP-seq signal per enclave is performed) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Used to aggregate mean ChIP-seq signal per bottom enclave and prepare ranked gene lists)

## Examples

```
chr_dat <- read_delim('./path/to/hic_file.txt'); chr_mat <- Rao_matrix(chr_dat); chr_pow <- pow_trans(chr_mat,1); g_chr1 <- graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res <- spec_bipart(chr_pow, g_chr1)
```

## Evaluation signals

- RNAP2-based GSEA in IMR90 reproduces enrichment of skin function, structural molecule activity (cytoskeleton) and DNA packaging GO categories
- H3K4me3-based GSEA in K562 reproduces enrichment of immune function GO categories
- Enrichment p-values are Benjamini-Hochberg FDR-corrected across all tested GO gene sets before reporting
- Every gene in the ranked list maps to exactly one bottom enclave in the BHi-Cect nested cluster list
- gsea_results.csv contains both FDR-corrected enrichment scores and the gene-set hits

## Limitations

- No software version stated for BHi-Cect or any analysis software
- No dataset accession IDs or named data repositories given for IMR90 and K562 ChIP-seq data; cell lines are referenced without accessions
- Enclave arrangements may vary across tissues; cross-tissue variability was not examined and flagged as future work
- The bottom-enclave epigenomic associations link nesting to activity but do not establish mechanism (e.g. loop extrusion vs phase separation)

## Evidence

- [discussion] GSEA on RNAP2 and H3K4me3 prevalence in bottom enclaves: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [discussion] Enrichment findings per cell line: "cytoskeleton and skin function enrichment in IMR90 fibroblast samples or immune function enrichment in K562 leukemia samples"
- [abstract] Enclaves as interwoven clusters: "the hierarchical structure of chromosome is organized as 'enclaves', which are complex interwoven clusters at both local and global scales"
- [readme] README spectral clustering invocation: "chr_spec_res<- spec_bipart(chr_pow ,g_chr1)"
- [readme] README required R packages: "install.packages(c('readr','caret','Matrix','igraph','RSpectra','dplyr','data.tree'))"
