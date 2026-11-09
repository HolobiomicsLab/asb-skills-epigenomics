---
name: chipseq-signal-aggregation-per-cluster
description: Use when BHi-Cect tree output (nested cluster list) and ChIP-seq tracks
  must be combined to compute epigenomic signal prevalence per bottom enclave for
  gene ranking or enrichment analysis.
when_to_use_negative:
- The clustering structure is not yet available — raw Hi-C alone cannot assign loci
  to bottom enclaves
- The analysis targets top-level/TAD-like contiguous domains rather than BHi-Cect
  bottom enclaves, which are mostly non-contiguous and deeply interwoven
- The goal is TF-content profiling of bottom enclaves (correspondence analysis), not
  ChIP-seq signal aggregation for GSEA
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3678
tools:
- name: BHi-Cect
  role: Produces the nested enclave tree and the named list of loci per cluster that
    define the aggregation units (bottom enclaves)
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: R
  role: Environment in which BHi-Cect functions are copy-pasted and the aggregation/GSEA
    pipeline is run
  repo: https://github.com/princeps091-binf/BHi-Cect
- name: dplyr
  role: Aggregates mean ChIP-seq signal per enclave per cell line before gene ranking
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkaa004
    title: 'BHi-Cect: a top-down algorithm for identifying the multi-scale hierarchical
      structure of chromosomes'
  source_content_hash: sha256:91eb63c36e0c8605ca88e0b937caa13672f251160d4f2563a009b0eb34ea1dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/princeps091-binf/BHi-Cect
  tools:
  - BHi-Cect
  - R
  - dplyr
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# chipseq-signal-aggregation-per-cluster

## Summary

Aggregates mean ChIP-seq signal (e.g., RNAP2 in IMR90, H3K4me3 in K562) over BHi-Cect bottom-enclave loci to produce per-enclave prevalence values that are used to rank genes for downstream GO GSEA. It connects Hi-C-derived enclave structure with epigenomic activity of the underlying DNA.

## When to use

Use when BHi-Cect tree output (nested cluster list) and ChIP-seq tracks must be combined to compute epigenomic signal prevalence per bottom enclave for gene ranking or enrichment analysis.

## When NOT to use

- The clustering structure is not yet available — raw Hi-C alone cannot assign loci to bottom enclaves
- The analysis targets top-level/TAD-like contiguous domains rather than BHi-Cect bottom enclaves, which are mostly non-contiguous and deeply interwoven
- The goal is TF-content profiling of bottom enclaves (correspondence analysis), not ChIP-seq signal aggregation for GSEA

## Inputs

- 3-column sparse Hi-C contact matrix per chromosome (e.g., chr17/chr22/chr18 IMR90)
- BHi-Cect tree output: nested cluster list plus named list of loci per cluster (bottom enclaves)
- ChIP-seq signal tracks: RNAP2 for IMR90, H3K4me3 for K562
- Gene-to-genomic-locus mapping

## Outputs

- Per-bottom-enclave mean ChIP-seq prevalence values (RNAP2 IMR90; H3K4me3 K562)
- Per-enclave ranked gene lists
- FDR-corrected GO GSEA enrichment scores written to gsea_results.csv

## How to apply

First obtain the BHi-Cect clustering output — a list with a nested list defining the tree and a named list specifying the loci contained in every cluster — by loading the 3-column sparse Hi-C matrix, running Rao_matrix, pow_trans(mat,1), graph_from_adjacency_matrix, and spec_bipart. Assign each gene's genomic locus to its bottom enclave, then use dplyr in R to aggregate mean ChIP-seq signal per enclave separately for RNAP2 (IMR90) and H3K4me3 (K562). Rank genes by aggregated signal intensity within each bottom enclave and run GSEA with GO gene sets, applying Benjamini-Hochberg FDR correction across all tested gene sets. The rationale is the paper's finding that nesting of local clusters within global clusters (enclaves) is associated with the epigenomic activity of the underlying DNA, so bottom-enclave prevalence is an enrichment signal. Correctness is judged by reproducing the reported enrichment patterns: skin function, cytoskeleton/structural molecule activity, and DNA packaging for IMR90 RNAP2; immune functions for K562 H3K4me3.

## Related tools

- **BHi-Cect** (Produces the nested enclave tree and the named list of loci per cluster that define the aggregation units (bottom enclaves)) — https://github.com/princeps091-binf/BHi-Cect
- **R** (Environment in which BHi-Cect functions are copy-pasted and the aggregation/GSEA pipeline is run) — https://github.com/princeps091-binf/BHi-Cect
- **dplyr** (Aggregates mean ChIP-seq signal per enclave per cell line before gene ranking)

## Examples

```
chr_dat<-read_delim('./hic_file.txt'); chr_mat<-Rao_matrix(chr_dat); chr_pow<-pow_trans(chr_mat,1); g_chr<-graph_from_adjacency_matrix(chr_pow,weighted=T); chr_spec_res<-spec_bipart(chr_pow,g_chr)
```

## Evaluation signals

- Reproduced enrichment categories match the report: skin function, structural molecule activity (cytoskeleton) and DNA packaging for IMR90 RNAP2; immune functions for K562 H3K4me3
- Benjamini-Hochberg FDR correction applied across all tested GO gene sets before interpretation
- Every gene in the ranked list maps to exactly one bottom enclave in the named locus list from the BHi-Cect output
- gsea_results.csv contains FDR-corrected enrichment scores and gene-set hits for both the RNAP2 (IMR90) and H3K4me3 (K562) tracks

## Limitations

- No dataset accession IDs or named data repositories are given for the IMR90 and K562 ChIP-seq inputs in the discussion section
- Cross-tissue variability of enclave arrangement was not examined; flagged as future work by the authors
- BHi-Cect is R-script based (functions to copy-paste); a stand-alone package implementation was still ongoing at the time of the README
- With low resolution Hi-C data (50 kb) BHi-Cect highlights global chromosome clusters, whereas with high resolution (5 kb) data it focuses on local clusters, affecting which enclaves are identified

## Evidence

- [abstract] Enclave-epigenomic association rationale: "We show that the nesting of local clusters within global clusters characterizing enclaves, is associated with the epigenomic activity found on the underlying DNA."
- [discussion] GSEA on RNAP2/H3K4me3 prevalence in bottom enclaves: "GSEA results focusing on the prevalence of RNAP2 and H3K4me3 in bottom enclaves"
- [readme] BHi-Cect output structure used to define aggregation units: "a namesd list specifying the loci contained by all the clusters found by BHi-Cect"
- [discussion] Enclaves are non-contiguous clusters, so locus assignment cannot assume genomic contiguity: "enclaves, which are a novel form of chromosome clusters identified by our method, that are mostly non-contiguous and deeply interwoven"
- [discussion] Resolution dependence of clusters found: "with low resolution Hi-C data (50 kb), BHi-Cect highlights global clusters spanning the whole chromosome while with high resolution Hi-C data (5 kb)"
