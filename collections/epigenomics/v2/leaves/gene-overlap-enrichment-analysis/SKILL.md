---
name: gene-overlap-enrichment-analysis
description: Use when you have two gene/region sets (e.g., differentially modified
  regions from ChIP-seq and differentially expressed genes from RNA-seq) mapped to
  a common identifier and need to quantify overlap significance and GO enrichment.
when_to_use_negative:
- You need to detect differentially modified regions themselves rather than test overlap
  of existing calls — use histoneHMM's bivariate HMM workflow instead
- Peak-like histone marks (narrow punctate ChIP-seq signals) rather than broad modifications
  like H3K27me3/H3K9me3 are the input, since competing peak-caller-based region sets
  behave differently on broad marks
- Only one gene set exists (no DE gene list), making a 2x2 contingency table undefined
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: histoneHMM
  role: Produces the differentially modified region calls (get.differential.regions)
    that are tested for overlap with DE genes
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment for building contingency tables, running stats::fisher.test,
    and writing fisher_overlap_pvalues.csv
- name: Bioconductor
  role: Provides the bioinformatic tool sets (e.g., genomic ranges/annotation handling)
    that histoneHMM integrates with for identifier mapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:3e0c998e9f0b094072c363b8bf73a8d9abee4819c95165c045fe2bf62fd13781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-overlap-enrichment-analysis

## Summary

Statistical testing of whether differentially modified histone regions overlap with differentially expressed genes more than expected, using 2x2 contingency tables and Fisher's exact test, followed by GO enrichment of the overlapping gene set. This is the validation strategy the histoneHMM paper used to show its calls are functionally relevant versus competing methods.

## When to use

Use when you have two gene/region sets (e.g., differentially modified regions from ChIP-seq and differentially expressed genes from RNA-seq) mapped to a common identifier and need to quantify overlap significance and GO enrichment.

## When NOT to use

- You need to detect differentially modified regions themselves rather than test overlap of existing calls — use histoneHMM's bivariate HMM workflow instead
- Peak-like histone marks (narrow punctate ChIP-seq signals) rather than broad modifications like H3K27me3/H3K9me3 are the input, since competing peak-caller-based region sets behave differently on broad marks
- Only one gene set exists (no DE gene list), making a 2x2 contingency table undefined

## Inputs

- RNA-seq gene expression table (expression.txt) for deriving DE genes at DESeq FDR<0.01
- histoneHMM differential modification calls (rat.H3K27me3 and mouse H3K9me3), regenerated with get.differential.regions if needed
- Gene annotation in GFF format (ensembl59-genes.gff) for identifier mapping

## Outputs

- 2x2 contingency tables of DE x differentially-modified overlap for rat and mouse comparisons
- Fisher's exact test P-values for each overlap test
- GO enrichment P-value for term GO:0019882 on the rat overlap gene set
- CSV file (fisher_overlap_pvalues.csv) with one P-value row per test

## How to apply

Derive the differentially expressed gene list from RNA-seq data (expression.txt) at DESeq FDR<0.01, and the differentially modified region list from histoneHMM (get.differential.regions) for the rat SHR/BN H3K27me3 and mouse H3K9me3 comparisons. Map both gene sets to a common identifier using the gene annotation in GFF format (ensembl59-genes.gff, gene bodies only, no exon information), then build 2x2 contingency tables of DE status versus differential modification status. Run Fisher's exact test (R stats::fisher.test) on each table and compare P-values against reported values of 3.36e-6 (rat H3K27me3) and 1.39e-6 (mouse H3K9me3); histoneHMM should yield the most significant overlap relative to competing methods. Finally, run a GO enrichment test on the overlapping rat gene set and verify enrichment of 'antigen processing and presentation' (GO:0019882) at P=4.79e-7.

## Related tools

- **histoneHMM** (Produces the differentially modified region calls (get.differential.regions) that are tested for overlap with DE genes) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment for building contingency tables, running stats::fisher.test, and writing fisher_overlap_pvalues.csv)
- **Bioconductor** (Provides the bioinformatic tool sets (e.g., genomic ranges/annotation handling) that histoneHMM integrates with for identifier mapping)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM")
```

## Evaluation signals

- Recomputed Fisher's exact P-values reproduce the reported values 3.36e-6 (rat H3K27me3) and 1.39e-6 (mouse H3K9me3) at least to order of magnitude
- GO:0019882 ('antigen processing and presentation') enrichment P-value matches the reported 4.79e-7
- fisher_overlap_pvalues.csv exists and contains one P-value row per test (rat overlap, mouse overlap, GO enrichment)
- histoneHMM's overlap P-values are more significant than those from competing methods' region calls (Figure 3b comparison)
- Both gene sets were successfully mapped to a common identifier via ensembl59-genes.gff, yielding non-degenerate 2x2 contingency tables

## Limitations

- Acceptance allows only order-of-magnitude agreement of overlap P-values (3.36e-6, 1.39e-6); exact reproduction may depend on histoneHMM version and region regeneration
- The ensembl59-genes.gff annotation contains only gene bodies with no exon information, limiting region-to-gene assignment granularity
- No version number or archival DOI is stated for histoneHMM, so reproducing region calls may be sensitive to package version (v1.7+ required for Rcpp >0.12 compatibility)

## Evidence

- [results] histoneHMM yielded the most significant overlap between differentially modified and differentially expressed genes (P=3.36×10−6, Fisher's exact test, Figure 3b): [evidence span withheld — anchor status: not_found]
- [results] gene ontology analysis revealed enrichment for the GO term antigen processing and presentation (GO:0019882, P=4.79·10−7): [evidence span withheld — anchor status: not_found]
- [readme] ensembl59-genes.gff : gene annotation in gff format (only gene bodies, no exon information): "ensembl59-genes.gff : gene annotation in gff format (only gene bodies, no exon information)"
- [intro] we show that histoneHMM outperforms competing methods in calling differentially modified regions between samples: "we show that histoneHMM outperforms competing methods in calling differentially modified regions between samples."
- [methods] install.packages("devtools") devtools::install_github("matthiasheinig/histoneHMM"): "install.packages("devtools")
devtools::install_github("matthiasheinig/histoneHMM")"
