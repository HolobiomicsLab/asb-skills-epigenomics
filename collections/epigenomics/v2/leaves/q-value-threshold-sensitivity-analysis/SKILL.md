---
name: q-value-threshold-sensitivity-analysis
description: Use when you must justify the FIMO q-value cutoff for motif-annotating
  CRE sequences, or test whether removing overlapping/redundant motif annotations
  degrades cell-type-specific classification performance.
when_to_use_negative:
- When the input is already a motif-count feature table — the analysis requires raw
  CRE coordinates and re-running FIMO at each q-value cutoff
- When the goal is de novo motif discovery rather than sensitivity of annotation-based
  classification (GimmeMotifs' prediction pipeline is a different tool)
- When regulatory activity depends on information beyond the input sequence (open
  chromatin, distal influences), since BOM is constrained to cis-regulatory information
  within the input sequence
edam_operation: http://edamontology.org/operation_0300
edam_topics:
- http://edamontology.org/topic_3511
tools:
- name: FIMO
  role: Scans CRE sequences for TF motifs, re-run at q-value cutoffs 0.1, 0.3, and
    0.5 to vary annotation stringency
  repo: https://meme-suite.org/meme/doc/install.html
- name: BagOfMotifs (BOM)
  role: Wraps runFIMO for scanning, converts non-overlapping annotations to motif-count
    features, trains XGBoost models (binModel) and evaluates auROC (predict_binary_multi)
  repo: github.com/ewonglab/BOM_package
- name: BSgenome.Mmusculus.UCSC.mm10
  role: Provides the mouse mm10 genome sequence used to extract CRE FASTA for motif
    scanning
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:0ca8182a3850a62f11f14d2898308d6e1bc7d45e14bafc279b0b335b37fd536b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Mmusculus.UCSC.mm10
  - BagOfMotifs
  - BagOfMotifs (BOM)
  - FIMO
  - binModel
  edam_operation: http://edamontology.org/operation_0300
  edam_topics:
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# q-value-threshold-sensitivity-analysis

## Summary

Sensitivity analysis over FIMO motif-scanning q-value cutoffs (0.1, 0.3, 0.5) to measure how motif-annotation coverage and downstream BOM auROC depend on annotation stringency, including testing whether overlapping (seemingly redundant) motif annotations carry biologically meaningful information.

## When to use

Use when you must justify the FIMO q-value cutoff for motif-annotating CRE sequences, or test whether removing overlapping/redundant motif annotations degrades cell-type-specific classification performance.

## When NOT to use

- When the input is already a motif-count feature table — the analysis requires raw CRE coordinates and re-running FIMO at each q-value cutoff
- When the goal is de novo motif discovery rather than sensitivity of annotation-based classification (GimmeMotifs' prediction pipeline is a different tool)
- When regulatory activity depends on information beyond the input sequence (open chromatin, distal influences), since BOM is constrained to cis-regulatory information within the input sequence

## Inputs

- Four-column CRE coordinate files (chromosome, start, stop, condition) for mouse E8.25 and human cell-line datasets
- gimme.vertebrate.v5.0.meme motif library (shipped with the BagOfMotifs package)
- Mouse mm10 genome via BSgenome.Mmusculus.UCSC.mm10
- Locally installed FIMO binary (path supplied as FIMO_path)

## Outputs

- FIMO motif annotations at q-value cutoffs 0.1, 0.3, and 0.5
- Per-threshold fraction of CRE nucleotides covered by at least one motif annotation
- auroc_sensitivity_results.csv with per-dataset auROC values before and after retraining on non-overlapping motif annotations

## How to apply

Generate CRE FASTA files from four-column BED-like inputs (chromosome, start, stop, condition) with generateAllFasta against the mm10 genome, then scan sequences with BagOfMotifs::runFIMO using the gimme.vertebrate.v5.0.meme motif library and a local FIMO installation. Repeat the scan at q-value cutoffs 0.1, 0.3, and 0.5; at each threshold compute the fraction of CRE nucleotides covered by at least one motif annotation and verify the annotation fraction reaches approximately 83% at q<=0.5. Convert non-overlapping motif annotations into per-sequence motif counts, train gradient-boosted tree classifiers with BagOfMotifs::binModel(), and evaluate cell-type-specific prediction with BagOfMotifs::predict_binary_multi(). Compare auROC values before and after retraining on non-overlapping annotations, saving per-dataset results to auroc_sensitivity_results.csv; the expected outcome is that removing overlapping annotations reduces auROC by about 0.32, indicating overlapping motif annotation may reflect composite binding sites.

## Related tools

- **FIMO** (Scans CRE sequences for TF motifs, re-run at q-value cutoffs 0.1, 0.3, and 0.5 to vary annotation stringency) — https://meme-suite.org/meme/doc/install.html
- **BagOfMotifs (BOM)** (Wraps runFIMO for scanning, converts non-overlapping annotations to motif-count features, trains XGBoost models (binModel) and evaluates auROC (predict_binary_multi)) — github.com/ewonglab/BOM_package
- **BSgenome.Mmusculus.UCSC.mm10** (Provides the mouse mm10 genome sequence used to extract CRE FASTA for motif scanning)

## Examples

```
BagOfMotifs::runFIMO(motifs_path = motifs_path, FIMO_path = FIMO_path)
```

## Evaluation signals

- At q<=0.5, the fraction of CRE nucleotides covered by at least one motif annotation reaches approximately 83%
- Annotation coverage increases monotonically as the q-value cutoff is relaxed from 0.1 to 0.5
- Retraining on non-overlapping motif annotations reproduces the reported 0.32 auROC reduction relative to the original motif-annotation models
- auroc_sensitivity_results.csv contains per-dataset auROC values for both the original and non-overlapping-annotation models across the mouse E8.25 and human cell-line datasets

## Limitations

- TFs do not bind all their motifs, so motif annotation presence does not guarantee binding
- Reliance on known TF motifs neglects informative sequences not captured by current motif annotations (e.g., the gimme.vertebrate.v5.0 library)
- BOM is constrained to cis-regulatory information within the input sequence; open chromatin status, histone marks, and enhancer activity can be influenced by regions beyond the sequence length used
- The ~83% coverage figure is an approximate check at q<=0.5; other q-value cutoffs require their own coverage verification

## Evidence

- [other] Repeat the FIMO scans at q-value cutoffs 0.1, 0.3, and 0.5, and at each threshold compute the fraction of CRE nucleotides covered: [evidence span withheld — anchor status: not_found]
- [other] Removing overlapping motif annotations reduced BOM's auROC by 0.32: [evidence span withheld — anchor status: not_found]
- [methods] runFIMO call from methods: "BagOfMotifs::runFIMO(
	motifs_path = motifs_path,
	FIMO_path   = FIMO_path
)"
- [readme] mm10 genome dependency from readme: "The tutorial requires the mouse mm10 genome. If you don't have this already:

r
BiocManager::install("BSgenome.Mmusculus.UCSC.mm10")"
