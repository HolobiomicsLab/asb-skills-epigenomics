---
name: h3k27ac-bigwig-enhancer-activity-quantification
description: Use when you have enhancer interval BED coordinates (e.g. from rearranged
  regulatory domains) plus control and patient/condition H3K27ac bigWig tracks for
  a phenotype-matched cell type and need per-enhancer mean activity values to score
  enhancer gain/loss.
when_to_use_negative:
- When the phenotype-relevant cell type or developmental stage lacks suitable bulk
  H3K27ac data — e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so
  POSTRE fails to predict the SHH duplication case
- When the SV effect is purely coding (gene deletion or truncation) rather than long-range
  enhancer-driven; enhancer activity quantification adds no signal there
edam_operation: http://edamontology.org/operation_3806
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: Pipeline that derives rearranged enhancer-gene domains from SV breakpoints
    and TAD boundaries and aggregates bigWig-derived enhancer activities into geneEnhancerScore_LOF/GOF
    labels
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function to batch-process multiple SVs, computing per-enhancer H3K27ac activity
    and writing enhancer_activity_scores.tsv
  repo: https://github.com/vicsanga/Postre
- name: bigWigAverageOverBed
  role: Extracts the mean H3K27ac signal per enhancer interval from control and patient
    bigWig tracks
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:e3693bacd247a140257f9b7148d19c3dd234036ba0bf28526ff1a0ffb94b1290
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/vicsanga/Postre
  repo_url: https://github.com/vicsanga/Postre
  tools:
  - POSTRE
  - POSTRE_multiSV()
  - R
  - bigWigAverageOverBed
  edam_operation: http://edamontology.org/operation_3806
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# h3k27ac-bigwig-enhancer-activity-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies per-enhancer H3K27ac signal by extracting the mean bigWig signal over each enhancer interval with bigWigAverageOverBed, separately for control and patient (or condition-matched) samples in the phenotype-relevant cell type, enabling detection of enhancer disconnection and enhancer adoption in structural variant analysis (as done in POSTRE).

## When to use

Use when you have enhancer interval BED coordinates (e.g. from rearranged regulatory domains) plus control and patient/condition H3K27ac bigWig tracks for a phenotype-matched cell type and need per-enhancer mean activity values to score enhancer gain/loss.

## When NOT to use

- When the phenotype-relevant cell type or developmental stage lacks suitable bulk H3K27ac data — e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case
- When the SV effect is purely coding (gene deletion or truncation) rather than long-range enhancer-driven; enhancer activity quantification adds no signal there

## Inputs

- Enhancer interval coordinates (BED-format intervals) within rearranged enhancer-gene domains derived from SV breakpoints and cell-type-specific TAD boundaries
- Control (wild-type) H3K27ac bigWig track for the phenotype-matched cell type
- Patient/condition H3K27ac bigWig track for the phenotype-matched cell type
- Structural variant file: plain text, 1 line and 7 tab-separated columns per SV (chr/breakpoint1, hg19 coordinates or comma-separated range, chr/breakpoint2, hg19 coordinates or range, SV type among Inversion/Translocation/Deletion/Duplication, comma-separated phenotype list, unique SV id)

## Outputs

- Per-enhancer mean H3K27ac values: enhancerActivityControl and enhancerActivityPatient (from bigWigAverageOverBed)
- Per-candidate-gene aggregated activity scores with binary geneEnhancerScore_LOF / geneEnhancerScore_GOF labels
- Tab-separated output table enhancer_activity_scores.tsv with columns: gene, phenotype, enhancerActivityControl, enhancerActivityPatient, geneEnhancerScore_LOF/GOF

## How to apply

First define the enhancer set from the affected regulatory domain: for strictly intrachromosomal SVs (deletions, inversions, duplications) order breakpoints so breakpoint 1 has the smaller hg19 coordinate, classify each breakpoint as intra- or inter-TAD against cell-type-specific TAD boundaries, and derive the rearranged enhancer-gene domain. Select the bigWig pair matching the phenotype (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary). For every enhancer interval in the affected domains, run bigWigAverageOverBed with the control bigWig to obtain enhancerActivityControl (mean H3K27ac per enhancer) and with the patient bigWig to obtain enhancerActivityPatient. Aggregate enhancer-level activity changes onto candidate genes within the rearranged domains to detect enhancer disconnection (activity loss) and enhancer adoption (activity gain). In POSTRE's scoring, a candidate gene receives geneEnhancerScore_LOF = 1 when enhancerActivityPatient_LOF is lower than enhancerActivityControl, and 0 otherwise.

## Related tools

- **POSTRE** (Pipeline that derives rearranged enhancer-gene domains from SV breakpoints and TAD boundaries and aggregates bigWig-derived enhancer activities into geneEnhancerScore_LOF/GOF labels) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function to batch-process multiple SVs, computing per-enhancer H3K27ac activity and writing enhancer_activity_scores.tsv) — https://github.com/vicsanga/Postre
- **bigWigAverageOverBed** (Extracts the mean H3K27ac signal per enhancer interval from control and patient bigWig tracks)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Every candidate gene row in enhancer_activity_scores.tsv carries exactly one binary LOF/GOF label
- Reproduce the benchmark case: POSTRE predicts loss of TFAP2A expression in neural crest cells through enhancer disconnection for the BOFS syndrome inversion patient (Laugsch et al., 2019)
- For genes scored geneEnhancerScore_LOF = 1, enhancerActivityPatient_LOF is lower than enhancerActivityControl; otherwise the score is 0
- bigWigAverageOverBed returns a finite mean signal for every enhancer interval in the affected domains from both control and patient bigWigs
- Input SV file parses correctly: 1 line and 7 tab-separated columns per SV, with SV type among Inversion/Translocation/Deletion/Duplication

## Limitations

- Prediction depends on the completeness of enhancer catalogs: incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms
- Bulk genomic data (e.g. whole limb bud) may miss locus-specific enhancers such as the ZRS, causing genes like SHH to appear inactive
- Lack of data for appropriate developmental stages or differentiation time points may affect identification of pathologically relevant genes
- POSTRE is oriented towards dominant disorders; capacity to identify SVs implicated in recessive phenotypes has not been specifically evaluated

## Evidence

- [methods] Compute enhancerIndividualActivity for every enhancer in the affected domains with bigWigAverageOverBed, extracting mean H3K27ac signal per enhancer interval from the control bigWig (enhancerActivityControl) and the patient bigWig: [evidence span withheld — anchor status: not_found]
- [other] POSTRE assigns geneEnhancerScoreLOF = 1 when enhancerActivityPatientLOF is lower than enhancerActivityControl: [evidence span withheld — anchor status: not_found]
- [readme] POSTRE successfully predicts the loss of TFAP2A expression in neural crest cells through an enhancer disconnection mechanism: [evidence span withheld — anchor status: not_found]
