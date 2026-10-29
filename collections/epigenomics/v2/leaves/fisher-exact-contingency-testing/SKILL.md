---
name: fisher-exact-contingency-testing
description: Use when you need to test enrichment of one genomic gene/region set (e.g.
  histoneHMM differential modification calls) against another (e.g. DESeq-derived
  DE genes) with a 2x2 contingency table and exact test P-value.
when_to_use_negative:
- Input is already a set of peak-like (narrow) ChIP-seq differential calls, since
  methods designed for peak-like features generate false positive or negative calls
  on broad marks like H3K27me3/H3K9me3
- The comparison involves ChIP-seq profiles from more than two samples, since histoneHMM's
  differential calling (and hence the contingency table) is defined for a two-sample
  contrast
- Expected overlap counts are so extreme that a chi-square test is appropriate and
  exact computation is unnecessary
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3295
tools:
- name: histoneHMM
  role: Produces the differentially modified region calls (get.differential.regions)
    whose overlap with DE genes is tested
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment in which stats::fisher.test and the GO enrichment test
    are run
- name: Bioconductor
  role: Ecosystem histoneHMM integrates with for genomic interval handling and annotation
    mapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:d5e74d739899a626547302796c115e9eb5f924acf54a9890bab2e4833f9b11ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# fisher-exact-contingency-testing

## Summary

Quantify whether differentially modified regions from histoneHMM overlap differentially expressed genes more than expected by chance, using 2x2 contingency tables and Fisher's exact test in R (stats::fisher.test). This is the validation approach the histoneHMM paper used to show histoneHMM outperforms competing methods on broad marks like H3K27me3 and H3K9me3.

## When to use

Use when you need to test enrichment of one genomic gene/region set (e.g. histoneHMM differential modification calls) against another (e.g. DESeq-derived DE genes) with a 2x2 contingency table and exact test P-value.

## When NOT to use

- Input is already a set of peak-like (narrow) ChIP-seq differential calls, since methods designed for peak-like features generate false positive or negative calls on broad marks like H3K27me3/H3K9me3
- The comparison involves ChIP-seq profiles from more than two samples, since histoneHMM's differential calling (and hence the contingency table) is defined for a two-sample contrast
- Expected overlap counts are so extreme that a chi-square test is appropriate and exact computation is unnecessary

## Inputs

- RNA-seq gene expression table (expression.txt) for deriving DE genes at DESeq FDR<0.01
- histoneHMM differential modification calls for rat H3K27me3 (SHR/BN) and mouse H3K9me3 comparisons
- Gene annotation in GFF format (ensembl59-genes.gff, gene bodies only, no exon information) for identifier mapping

## Outputs

- 2x2 contingency tables of DE x differentially-modified overlap for rat and mouse comparisons
- Fisher's exact test P-values per comparison (target values: 3.36e-6 rat H3K27me3, 1.39e-6 mouse H3K9me3)
- GO enrichment P-value for GO:0019882 on the rat overlap gene set (target: 4.79e-7)
- fisher_overlap_pvalues.csv containing one P-value row per test

## How to apply

Load the RNA-seq expression data (expression.txt) and derive the differentially expressed gene list at DESeq FDR<0.01. Load the histoneHMM differential modification calls (e.g. rat.H3K27me3 and mouse H3K9me3), regenerating them with get.differential.regions from the histoneHMM R package if needed. Map both gene sets to a common identifier via the GFF gene annotation (ensembl59-genes.gff), then build 2x2 contingency tables of differential-expression x differential-modification overlap for each comparison. Run stats::fisher.test on each table and compare P-values against reported values: 3.36e-6 for rat H3K27me3 and 1.39e-6 for mouse H3K9me3 (order-of-magnitude agreement). Optionally extend to GO enrichment testing, e.g. term GO:0019882 ('antigen processing and presentation') on the rat overlap gene set, expected P=4.79e-7.

## Related tools

- **histoneHMM** (Produces the differentially modified region calls (get.differential.regions) whose overlap with DE genes is tested) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment in which stats::fisher.test and the GO enrichment test are run)
- **Bioconductor** (Ecosystem histoneHMM integrates with for genomic interval handling and annotation mapping)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); fisher.test(table(de_gene, diff_modified))
```

## Evaluation signals

- Recomputed Fisher's exact P-value for rat H3K27me3 agrees with 3.36e-6 to at least order-of-magnitude
- Recomputed Fisher's exact P-value for mouse H3K9me3 agrees with 1.39e-6 to at least order-of-magnitude
- GO:0019882 enrichment P-value on the rat overlap gene set matches 4.79e-7
- fisher_overlap_pvalues.csv exists and contains exactly one P-value row per test (rat, mouse, GO)
- Gene sets from expression.txt and histoneHMM calls map to a consistent identifier space via ensembl59-genes.gff before table construction (non-zero overlap counts)

## Limitations

- histoneHMM is distributed without an archival version/DOI statement in the article, so exact software version used may be hard to pin (README notes latest version 1.7 requires Rcpp > 0.12)
- Supplementary 'Additional file 1' contents (e.g. simulation settings supporting parameter choices) are not included in the provided text
- Identifier mapping relies on gene-body-only GFF annotation (ensembl59-genes.gff has no exon information), which can affect gene-level assignment of differentially modified regions

## Evidence

- [results] P-value rat H3K27me3 overlap: [evidence span withheld — anchor status: not_found]
- [results] GO term enrichment: [evidence span withheld — anchor status: not_found]
- [results] mouse replication: [evidence span withheld — anchor status: not_found]
- [methods] package install: "devtools::install_github("matthiasheinig/histoneHMM")"
- [readme] gene annotation file description: "ensembl59-genes.gff : gene annotation in gff format (only gene bodies, no exon information)"
