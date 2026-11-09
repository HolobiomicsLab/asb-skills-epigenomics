---
name: kmer-svm-classifier-training
description: Use when benchmarking sequence-based classifiers on binary CRE-vs-background
  tasks (e.g., mouse E8.25 distal CREs) and a k-mer kernel SVM baseline such as LS-GKM
  is required.
when_to_use_negative:
- The goal is direct motif-level interpretability without extra annotation steps —
  k-mer SVMs discover sequence patterns but 'require additional motif annotation',
  whereas BOM yields motif counts and SHAP scores directly
- Input is already a motif-count feature table, in which case training XGBoost (BOM's
  binModel) on the table is the appropriate path rather than re-deriving k-mer features
  from sequence
- The task requires modeling motif order, orientation, or spacing — the k-mer kernel
  captures local k-mer composition but unordered-count alternatives like BOM are the
  intended comparison in this context
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3169
tools:
- name: LS-GKM
  role: k-mer SVM classifier trained with default gkm-SVM kernel settings on binary
    cell-type-specific CRE labels as the sequence-based baseline
- name: gkmSVM
  role: underlying gkm-SVM kernel framework used by LS-GKM for k-mer–based classification
- name: BagOfMotifs (BOM)
  role: motif-count XGBoost classifier against which the k-mer SVM baseline is benchmarked
    using auPR and MCC
  repo: https://github.com/ewonglab/BOM_package
- name: FIMO
  role: motif scanning used in the complementary BOM workflow; k-mer SVM results require
    additional motif annotation via tools like FIMO
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-65362-2
    title: Motif-based models accurately predict cell type-specific distal regulatory
      elements
  source_content_hash: sha256:bc8b41ad8bb71d2b34ecdcd7f4af2a246f56ce1f59433a6a6a5e78f6e9600bdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ewonglab/BOM_package
  tools:
  - BagOfMotifs (BOM)
  - DNABERT
  - FIMO
  - LS-GKM
  - gkmSVM
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kmer-svm-classifier-training

## Summary

Train gkmSVM/LS-GKM k-mer–based SVM classifiers on cell-type-specific distal cis-regulatory element (CRE) labels to serve as sequence-based baselines that can discover novel sequence patterns without prior motif annotation.

## When to use

Use when benchmarking sequence-based classifiers on binary CRE-vs-background tasks (e.g., mouse E8.25 distal CREs) and a k-mer kernel SVM baseline such as LS-GKM is required.

## When NOT to use

- The goal is direct motif-level interpretability without extra annotation steps — k-mer SVMs discover sequence patterns but 'require additional motif annotation', whereas BOM yields motif counts and SHAP scores directly
- Input is already a motif-count feature table, in which case training XGBoost (BOM's binModel) on the table is the appropriate path rather than re-deriving k-mer features from sequence
- The task requires modeling motif order, orientation, or spacing — the k-mer kernel captures local k-mer composition but unordered-count alternatives like BOM are the intended comparison in this context

## Inputs

- BED file with four columns (chromosome, start, stop, condition) defining cell-type-specific distal CREs and backgrounds
- FASTA sequences extracted from the mouse mm10 genome (BSgenome.Mmusculus.UCSC.mm10)
- Binary cell-type-specific CRE labels and train/test split definitions

## Outputs

- Trained LS-GKM/gkmSVM k-mer SVM models per cell type
- Held-out auPR and MCC metrics per cell type for baseline comparison
- Relative-improvement percentages versus other classifiers

## How to apply

Generate FASTA sequences from the input BED file (columns: chromosome, start, stop, condition) with generateAllFasta using genome 'Mmusculus' (mm10, BSgenome.Mmusculus.UCSC.mm10) so all models share identical training/testing sequences. Train LS-GKM with default gkm-SVM kernel settings on the same binary cell-type-specific CRE labels and train/test splits used by the other classifiers. Evaluate on the held-out set with auPR and MCC, then compute relative improvement against the k-mer SVM — in the source study BOM outperformed LS-GKM by 17.2% auPR and 77.5% MCC across 17 E8.25 cell types. k-mer–based classifiers can discover novel sequence patterns but require additional motif annotation, which is why they are compared against motif-count models like BOM.

## Related tools

- **LS-GKM** (k-mer SVM classifier trained with default gkm-SVM kernel settings on binary cell-type-specific CRE labels as the sequence-based baseline)
- **gkmSVM** (underlying gkm-SVM kernel framework used by LS-GKM for k-mer–based classification)
- **BagOfMotifs (BOM)** (motif-count XGBoost classifier against which the k-mer SVM baseline is benchmarked using auPR and MCC) — https://github.com/ewonglab/BOM_package
- **FIMO** (motif scanning used in the complementary BOM workflow; k-mer SVM results require additional motif annotation via tools like FIMO)

## Examples

```
generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")  # then train LS-GKM with default gkm-SVM kernel settings on the resulting FASTA and binary CRE labels
```

## Evaluation signals

- Computed auPR/MCC on the held-out set reproduce the reported relative improvements: BOM beats LS-GKM by 17.2% auPR and 77.5% MCC across the 17 E8.25 cell types
- All classifiers (LS-GKM, DNABERT, Enformer, CNN variants) are trained and evaluated on identical FASTA sequences generated from the same input BED file and mm10 genome
- Binary labels and train/test partitions are shared across all baseline models so metrics are directly comparable
- MCC is reported alongside auPR, since MCC improvements over LS-GKM (77.5%) exceed auPR improvements (17.2%)

## Limitations

- k-mer–based classifiers (gkmSVM, LS-GKM) can discover novel sequence patterns but require additional motif annotation for interpretation
- Deep-learning/k-mer baseline performance is likely limited by the amount of training data available for many cell states
- Like BOM, sequence-only classifiers are constrained to cis-regulatory information within the input sequence and cannot capture open chromatin status or histone marks

## Evidence

- [intro] K-mer–based classifiers (gkmSVM11, LS-GKM12) can discover novel sequence patterns but require additional motif annotation13,14.: "K-mer–based classifiers (gkmSVM11, LS-GKM12) can discover novel sequence patterns but require additional motif annotation13,14."
- [other] Train LS-GKM with default gkm-SVM kernel settings on the same binary cell-type-specific CRE labels and splits.: [evidence span withheld — anchor status: not_found]
- [other] BOM outperformed LS-GKM, DNABERT, and Enformer by 17.2%, 55.1%, and 10.3% in auPR and by 77.5%, 211.9%, and 33.4% in MCC, respectively.: [evidence span withheld — anchor status: not_found]
- [methods] generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus"): "generateAllFasta(bedDir = "path/to/input", genome = "Mmusculus")"
- [readme] The tutorial requires the mouse mm10 genome. If you don't have this already:: "The tutorial requires the mouse mm10 genome. If you don't have this already:"
