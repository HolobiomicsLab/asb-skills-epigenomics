---
name: peak-gene-expression-concordance-analysis
description: Use when you have qbed insertion data from two biological conditions
  (e.g. male vs female GBM Brd4 single-cell calling cards) plus matched single-cell
  RNA expression, and need to test concordance of differential binding with nearby
  gene expression.
when_to_use_negative:
- Input is already a processed peak-to-gene assignment table or feature matrix — the
  skill's value is in going from raw qbed insertions to concordant pairs.
- Data is bulk ChIP-seq or other non-calling-cards binding assay where the qbed/CCcaller/Mudata
  pipeline does not apply.
- No matched RNA expression data exist for the same cells, since pair_peak_gene_sc
  requires both binding and expression layers.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3179
- http://edamontology.org/topic_0203
tools:
- name: Pycallingcards
  role: Python framework providing rd, pp, tl, and pl modules that execute the full
    peak-calling to peak-gene pairing workflow
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: CCcaller
  role: Peak caller used to pinpoint Brd4 binding sites from joint male/female insertions
    (maxbetween=1100)
  repo: https://github.com/The-Mitra-Lab/pycallingcards
- name: Scanpy
  role: Single-cell analysis environment with which Pycallingcards interacts; provides
    the gene-expression AnnData layer
- name: Mudata
  role: Combines the peak AnnData and gene-expression AnnData objects into one object
    for paired analysis
- name: bedtools
  role: pybedtools backend for preprocessing.annotation of called peaks against genomic
    features
  repo: https://bedtools.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae070
    title: 'Pycallingcards: an integrated environment for visualizing, analyzing,
      and interpreting Calling Cards data'
  source_content_hash: sha256:fd896ac199cd2941d427a97b370c01380a5ff55f38b450f4328a0f168a2b428c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/The-Mitra-Lab/pycallingcards
  tools:
  - CCcaller
  - Mudata
  - Pycallingcards
  - Python
  - Scanpy
  - bedtools
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3179
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# peak-gene-expression-concordance-analysis

## Summary

Pairs differentially called Calling Cards peaks (TF binding) with differential expression of nearby genes to test whether sex-biased or condition-biased binding is concordant with gene expression, using Pycallingcards' tools.DE_pvalue and tools.pair_peak_gene_sc on Mudata objects.

## When to use

Use when you have qbed insertion data from two biological conditions (e.g. male vs female GBM Brd4 single-cell calling cards) plus matched single-cell RNA expression, and need to test concordance of differential binding with nearby gene expression.

## When NOT to use

- Input is already a processed peak-to-gene assignment table or feature matrix — the skill's value is in going from raw qbed insertions to concordant pairs.
- Data is bulk ChIP-seq or other non-calling-cards binding assay where the qbed/CCcaller/Mudata pipeline does not apply.
- No matched RNA expression data exist for the same cells, since pair_peak_gene_sc requires both binding and expression layers.

## Inputs

- qbed files of male and female GBM Brd4 single-cell calling cards insertions (GEO GSE148448)
- matched single-cell gene-expression AnnData object
- called and annotated peak set (peak bed files from CCcaller)

## Outputs

- Fisher's exact-test differential peak table (tools.DE_pvalue)
- peak-gene pair concordance table with P-values (tools.pair_peak_gene_sc)
- volcano plots and dotplots reproducing Fig. 4a-g sex-biased loci

## How to apply

Load male and female GBM Brd4 single-cell qbed datasets (Moudgil et al., Cell 2020; GEO GSE148448) with pycallingcards reading (rd) functions and clean insertions via preprocessing.clean_qbed. Combine peaks across conditions with preprocessing.combine_peaks and call peaks on the joint data with preprocessing.call_peaks using the CCcaller peak caller (maxbetween=1100), then annotate peaks with preprocessing.annotation (bedtools/pybedtools backend) and build cell-by-peak AnnData matrices with preprocessing.make_Anndata, combining peak and gene-expression AnnData objects into a Mudata object. Run differential peak analysis between conditions with tools.DE_pvalue (Fisher's exact tests) to identify sex-biased binding sites, then pair differential peaks with gene expression using tools.pair_peak_gene_sc. Accept the reanalysis only if the differential binding table is produced, peak-gene concordance P-values for sex-biased pairs are < .001, and target loci (Sema3a, Adam19, Zic1 female-biased; Nkx2-1 male-biased) match the reported pattern (Fig. 4a-g), which can be visualized with plotting.volcano_plot and plotting.dotplot_sc.

## Related tools

- **Pycallingcards** (Python framework providing rd, pp, tl, and pl modules that execute the full peak-calling to peak-gene pairing workflow) — https://github.com/The-Mitra-Lab/pycallingcards
- **CCcaller** (Peak caller used to pinpoint Brd4 binding sites from joint male/female insertions (maxbetween=1100)) — https://github.com/The-Mitra-Lab/pycallingcards
- **Scanpy** (Single-cell analysis environment with which Pycallingcards interacts; provides the gene-expression AnnData layer)
- **Mudata** (Combines the peak AnnData and gene-expression AnnData objects into one object for paired analysis)
- **bedtools** (pybedtools backend for preprocessing.annotation of called peaks against genomic features) — https://bedtools.readthedocs.io/en/latest/

## Examples

```
import pycallingcards as cc; qbed_m = cc.rd.read_qbed('male_GBM_brd4.qbed'); qbed_f = cc.rd.read_qbed('female_GBM_brd4.qbed'); peaks = cc.pp.call_peaks(cc.pp.combine_peaks({"male": cc.pp.clean_qbed(qbed_m), "female": cc.pp.clean_qbed(qbed_f)}), method='CCcaller', maxbetween=1100); cc.tl.pair_peak_gene_sc(mdata, pvalue_cutoff=0.001)
```

## Evaluation signals

- Fisher's exact-test differential binding table (tools.DE_pvalue) is produced for male vs female peaks.
- Peak-gene concordance P-values for sex-biased pairs are < .001 (tools.pair_peak_gene_sc).
- Sema3a, Adam19, and Zic1 show more Brd4 binding and higher RNA expression in female cells; Nkx2-1 shows both in male cells.
- Volcano plot and dotplot outputs reproduce the reported Fig. 4a-g pattern.

## Limitations

- Annotation via preprocessing.annotation requires bedtools/pybedtools to be installed as an external dependency.
- Concordance is a hypothesis-generating signal ('potential hypothesis for the relationship between transcription factor bindings and gene expression'), not a causal proof.
- Results depend on the peak caller choice; CCcaller and MACCs differ in accuracy/speed for pinpointing TF binding sites.

## Evidence

- [methods] Differential peak analysis is then followed and pair analysis is completed to find out potential hypothesis: "Differential peak analysis is then followed and pair analysis is completed to find out potential hypothesis"
- [methods] pair differential peaks with gene expression: "**Tools (tl)** calls motif of the peaks, completes differential peaks and pair differential peaks with gene expression,"
- [methods] Two Anndata objects can combine into one Mudata object.: "Two Anndata objects can combine into one Mudata object."
- [abstract] This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators: "This analysis revealed novel cell-type-specific binding sites and potential sex-linked TF regulators, furthering our understanding of TF binding and gene expression relationships."
- [readme] demonstrate potenial binding-gene expression relationships: "display differential peaks, demonstrate potenial binding-gene expression relationships."
