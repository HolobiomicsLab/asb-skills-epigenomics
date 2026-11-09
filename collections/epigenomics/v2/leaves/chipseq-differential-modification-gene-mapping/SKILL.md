---
name: chipseq-differential-modification-gene-mapping
description: Use when ChIP-seq differential modification calls (e.g., broad marks
  like H3K27me3/H3K9me3) must be intersected with a DESeq-derived DE gene list (FDR<0.01)
  and the overlap significance quantified and compared across peak-calling methods.
when_to_use_negative:
- The modification is a narrow/peak-like mark rather than a broad footprint (H3K27me3/H3K9me3-style);
  the article notes peak-searching methods generate false positives/negatives on broad
  marks and histoneHMM was designed for this regime.
- No matched gene expression data exist, so a DE gene list (DESeq FDR<0.01) cannot
  be constructed for the overlap test.
- Samples are from different strains/species without a common gene identifier mapping
  strategy via the GFF annotation.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
tools:
- name: histoneHMM
  role: Bivariate HMM used to call differentially modified regions (get.differential.regions)
    whose gene overlap is tested
  repo: https://github.com/matthiasheinig/histoneHMM
- name: R
  role: Computing environment for running histoneHMM, building contingency tables,
    and stats::fisher.test
- name: Bioconductor
  role: Bioinformatic tool sets used for gene/region annotation mapping to a common
    identifier
- name: samtools
  role: Removes duplicated reads during ChIP-seq preprocessing before region calling
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-015-0491-6
    title: 'histoneHMM: Differential analysis of histone modifications with broad
      genomic footprints'
  source_content_hash: sha256:2e97566fe05bef1c8b68222914672b4dbd19188023ce94a913ee05001fb64c87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/matthiasheinig/histoneHMM
  tools:
  - Bioconductor
  - R
  - histoneHMM
  - samtools
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# chipseq-differential-modification-gene-mapping

## Summary

Tests whether histoneHMM's differentially modified ChIP-seq regions (e.g., rat SHR/BN H3K27me3, mouse H3K9me3) overlap differentially expressed genes more significantly than competing methods' calls, using Fisher's exact tests on 2x2 contingency tables built via GFF gene annotation mapping.

## When to use

Use when ChIP-seq differential modification calls (e.g., broad marks like H3K27me3/H3K9me3) must be intersected with a DESeq-derived DE gene list (FDR<0.01) and the overlap significance quantified and compared across peak-calling methods.

## When NOT to use

- The modification is a narrow/peak-like mark rather than a broad footprint (H3K27me3/H3K9me3-style); the article notes peak-searching methods generate false positives/negatives on broad marks and histoneHMM was designed for this regime.
- No matched gene expression data exist, so a DE gene list (DESeq FDR<0.01) cannot be constructed for the overlap test.
- Samples are from different strains/species without a common gene identifier mapping strategy via the GFF annotation.

## Inputs

- RNA-seq gene expression table (expression.txt)
- ChIP-seq signal/preprocessed read-count tables for the two compared samples (e.g., BN.txt, SHR.txt from histonehmm.molgen.mpg.de)
- Gene annotation in GFF format (ensembl59-genes.gff, gene bodies only)
- histoneHMM differential modification calls (rat H3K27me3 SHR/BN and mouse H3K9me3), via get.differential.regions

## Outputs

- 2x2 contingency tables of DE x differentially-modified overlap for rat and mouse
- Fisher's exact test P-values for each method/dataset combination (expected: 3.36e-6 rat H3K27me3, 1.39e-6 mouse H3K9me3)
- GO enrichment P-value for GO:0019882 on the rat overlap gene set (expected: 4.79e-7)
- CSV file (fisher_overlap_pvalues.csv) with one P-value row per test

## How to apply

Load the RNA-seq expression table (expression.txt) and derive the differentially expressed gene list at DESeq FDR<0.01. Call differentially modified regions with histoneHMM's get.differential.regions on the rat H3K27me3 (SHR vs BN) and mouse H3K9me3 datasets, regenerating region calls where needed. Map both the region-overlapping genes and the DE genes to a common identifier using the gene annotation in GFF format (ensembl59-genes.gff, gene bodies only), then build 2x2 contingency tables of DE x differentially-modified status. Run stats::fisher.test on each table and compare against the published overlap P-values of 3.36e-6 (rat H3K27me3) and 1.39e-6 (mouse H3K9me3); repeat for competing methods' calls to confirm histoneHMM yields the most significant overlap. Optionally run a GO enrichment test for GO:0019882 ('antigen processing and presentation') on the rat overlap gene set, expecting P=4.79e-7. Write all contingency tables and P-values to a single CSV, one row per test.

## Related tools

- **histoneHMM** (Bivariate HMM used to call differentially modified regions (get.differential.regions) whose gene overlap is tested) — https://github.com/matthiasheinig/histoneHMM
- **R** (Computing environment for running histoneHMM, building contingency tables, and stats::fisher.test)
- **Bioconductor** (Bioinformatic tool sets used for gene/region annotation mapping to a common identifier)
- **samtools** (Removes duplicated reads during ChIP-seq preprocessing before region calling)

## Examples

```
install.packages("devtools"); devtools::install_github("matthiasheinig/histoneHMM"); fisher.test(table(is_de, is_diff_mod))
```

## Evaluation signals

- Recomputed Fisher's exact P-value for rat H3K27me3 overlap agrees with 3.36e-6 (order-of-magnitude).
- Recomputed Fisher's exact P-value for mouse H3K9me3 overlap agrees with 1.39e-6 (order-of-magnitude).
- GO:0019882 enrichment P-value on the rat overlap gene set matches 4.79e-7.
- fisher_overlap_pvalues.csv exists and contains exactly one P-value row per test performed.
- histoneHMM's overlap P-value is more significant than those from competing methods' region calls (Diffreps, ChIPdiff, RSEG, SICER, PEP, BroadPeak).

## Limitations

- Overlap significance depends on the DE gene cutoff (DESeq FDR<0.01) and on gene-body mapping via ensembl59-genes.gff, which lacks exon information.
- Acceptance is order-of-magnitude agreement, so exact reproduction may vary with region-call regeneration and genome assembly choices (rat genome rn4; SHR reads aligned against BN reference with SHR SNP alleles substituted).
- The workflow presumes region calls can be regenerated with get.differential.regions; if raw BAM inputs are unavailable, only the published P-values can be cited.

## Evidence

- [results] histoneHMM yielded the most significant overlap between differentially modified and differentially expressed genes (P=3.36×10−6, Fisher's exact test, Figure 3b): [evidence span withheld — anchor status: not_found]
- [results] gene ontology analysis revealed enrichment for antigen processing and presentation: [evidence span withheld — anchor status: not_found]
- [other] Load the RNA-seq gene expression data (expression.txt) and derive the differentially expressed gene list at DESeq FDR<0.01: [evidence span withheld — anchor status: not_found]
- [readme] README lists expression.txt as RNA-seq gene expression and ensembl59-genes.gff as gene annotation in gff format (only gene bodies, no exon information): "[ensembl59-genes.gff](http://histonehmm.molgen.mpg.de/data/ensembl59-genes.gff) : gene annotation in gff format (only gene bodies, no exon information)"
- [methods] Installation via devtools from the matthiasheinig/histoneHMM GitHub repository: "devtools::install_github("matthiasheinig/histoneHMM")"
