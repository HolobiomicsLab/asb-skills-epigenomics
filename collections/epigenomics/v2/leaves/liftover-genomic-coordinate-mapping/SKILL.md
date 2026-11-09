---
name: liftover-genomic-coordinate-mapping
description: Use when motif-based (BOM) predictions transfer across species and you
  need to quantify how many predicted CREs lack detectable cross-species sequence
  alignment, as in mouse-trained models tested on human fetal CREs.
when_to_use_negative:
- The question is motif importance or SHAP-based interpretation within a single species/condition,
  not cross-species coordinate correspondence.
- Input is a motif count matrix or model training output rather than genomic coordinates
  of predicted elements.
- The CREs are known a priori to be sequence-homologous across species, so the alignment-dependence
  assessment is moot.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: UCSC liftOver
  role: Converts human CRE coordinates to mouse genome coordinates at minMatch 0.95
    and 0.6 to detect which predicted elements lack sequence alignment
- name: Bag-of-Motifs (BOM)
  role: Produces the correctly predicted human CRE coordinates via predict_binary_multi()
    that are subsequently liftOver-mapped
  repo: github.com/ewonglab/BOM_package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:fee08d1289bfe02cf747366ff54e49b790ddaa70484226ceb045a7a928c3cb54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bag-of-Motifs (BOM)
  - UCSC liftOver
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# liftover-genomic-coordinate-mapping

## Summary

Maps correctly predicted cross-species cis-regulatory elements between genomes (human-to-mouse) using UCSC liftOver at strict (minMatch 0.95) and relaxed (minMatch 0.6) alignment thresholds, to test whether motif-based BOM predictions depend on detectable mouse-human sequence alignment.

## When to use

Use when motif-based (BOM) predictions transfer across species and you need to quantify how many predicted CREs lack detectable cross-species sequence alignment, as in mouse-trained models tested on human fetal CREs.

## When NOT to use

- The question is motif importance or SHAP-based interpretation within a single species/condition, not cross-species coordinate correspondence.
- Input is a motif count matrix or model training output rather than genomic coordinates of predicted elements.
- The CREs are known a priori to be sequence-homologous across species, so the alignment-dependence assessment is moot.

## Inputs

- BED-format coordinates (chromosome, start, stop) of correctly predicted human fetal CREs from BagOfMotifs::predict_binary_multi() output
- Human-to-mouse liftOver chain file (e.g. hg38/hg19-to-mm10 chain, appropriate genome builds matching the CRE input)
- minMatch thresholds: 0.95 (strict) and 0.6 (relaxed)

## Outputs

- Lifted-over mouse coordinate BED file for correctly predicted CREs
- Unmapped/failed-liftOver element list at each minMatch threshold
- Alignment summary (fraction of correctly predicted CREs with no detectable mouse-human alignment) compiled into cross_species_predictions.csv

## How to apply

After BagOfMotifs::predict_binary_multi() classifies human fetal cardiomyocyte and erythroid CREs against mouse-trained BOM models, collect the coordinates of the correctly predicted elements (chromosome, start, stop BED format). Run UCSC liftOver with minMatch = 0.95 to map human coordinates to mouse, then rerun with minMatch = 0.6 as a relaxed retry; elements failing both passes count as lacking detectable mouse-human alignment. Compute the fraction of correctly predicted CREs with no alignment and combine it with the correct-classification rates (64.5% cardiomyocyte, 62.9% erythroblast) into cross_species_predictions.csv. Rationale: a high unaligned fraction demonstrates that BOM's gimme.vertebrate.v5.0.meme motif-count features (scanned by FIMO) capture a regulatory code beyond sequence homology.

## Related tools

- **UCSC liftOver** (Converts human CRE coordinates to mouse genome coordinates at minMatch 0.95 and 0.6 to detect which predicted elements lack sequence alignment)
- **Bag-of-Motifs (BOM)** (Produces the correctly predicted human CRE coordinates via predict_binary_multi() that are subsequently liftOver-mapped) — github.com/ewonglab/BOM_package

## Examples

```
liftOver -minMatch 0.95 predicted_human_cres.bed hg19ToMm10.over.chain.gz predicted_mm95.bed unmapped95.bed && liftOver -minMatch 0.6 predicted_human_cres.bed hg19ToMm10.over.chain.gz predicted_mm60.bed unmapped60.bed
```

## Evaluation signals

- Correct-classification rates on the human CREs should reproduce 64.5% (cardiomyocyte) and 62.9% (erythroblast) against the mouse-trained BOM models.
- At minMatch = 0.95, ~90.1% of correctly predicted cardiomyocyte CREs and ~96.6% of correctly predicted erythroblast CREs should fail liftOver mapping (no detectable mouse-human alignment).
- The relaxed minMatch = 0.6 pass should recover only a small additional set of aligned elements; the unmapped BED should remain consistent with the alignment summary.
- cross_species_predictions.csv must contain both the correct-classification rates and the liftOver alignment fractions, with row counts matching the number of correctly predicted CREs.

## Limitations

- BOM is constrained to cis-regulatory information within the input sequence length; lifted coordinates do not address regulatory activity driven by regions beyond the sequence used.
- liftOver minMatch thresholds determine alignment detectability — a strict 0.95 pass can overstate the 'no alignment' fraction, so the relaxed 0.6 retry is required before concluding absence of homology.
- The analysis does not determine how much of the remaining genome shares similar motif composition yet lacks regulatory activity.

## Evidence

- [results] Most correctly predicted elements lacked mouse-human alignment: [evidence span withheld — anchor status: not_found]
- [methods] Two-pass liftOver protocol: [evidence span withheld — anchor status: not_found]
- [readme] Genome build for sequence extraction: "library("BSgenome.Mmusculus.UCSC.mm10") # or genome of choice"
