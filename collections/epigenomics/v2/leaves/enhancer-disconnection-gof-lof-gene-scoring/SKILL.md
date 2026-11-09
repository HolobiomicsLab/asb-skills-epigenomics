---
name: enhancer-disconnection-gof-lof-gene-scoring
description: Use when a structural variant has been mapped to a cell-type-specific
  rearranged enhancer-gene domain and per-gene direction of enhancer activity change
  (loss vs gain) must be assigned from control vs patient H3K27ac bigWig signals.
when_to_use_negative:
- The SV effect is purely coding (gene deletion/truncation) rather than long-range
  enhancer-driven — use POSTRE's coding pathomechanism predictions instead
- The disease-relevant cell type lacks genomic data in POSTRE (e.g. bulk whole limb
  bud data cannot identify the ZRS enhancer, so SHH duplication cases fail)
- The phenotype of interest is recessive — POSTRE is oriented towards dominant disorders
  and its capacity for recessive phenotypes has not been specifically evaluated
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: POSTRE
  role: Computational framework whose POSTRE_multiSV() function derives the rearranged
    regulatory domains and computes the geneEnhancerScore_LOF/GOF labels per candidate
    gene
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function invoked on a 7-column SV data frame to run the per-SV and per-gene
    enhancer scoring pipeline and write the results tables
  repo: https://github.com/vicsanga/Postre
- name: bigWigAverageOverBed
  role: Extracts mean H3K27ac signal (enhancerActivityControl / enhancerActivityPatient)
    per enhancer interval in the affected domains
- name: XiaoTaoWang/domaincaller
  role: Directionality Index TAD caller whose TAD boundaries support the intra- vs
    inter-TAD breakpoint classification used to build rearranged domains
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:9d4c7f4431d048942ebddc0e7932568125b1e411a032a65be4b9d8205fab4ffb
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
  - XiaoTaoWang/domaincaller
  - bigWigAverageOverBed
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# enhancer-disconnection-gof-lof-gene-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates cell-type-specific H3K27ac enhancer activity changes across POSTRE-rearranged regulatory domains onto candidate genes, assigning each gene a binary geneEnhancerScore_LOF or geneEnhancerScore_GOF label that flags enhancer disconnection (loss) or enhancer adoption (gain).

## When to use

Use when a structural variant has been mapped to a cell-type-specific rearranged enhancer-gene domain and per-gene direction of enhancer activity change (loss vs gain) must be assigned from control vs patient H3K27ac bigWig signals.

## When NOT to use

- The SV effect is purely coding (gene deletion/truncation) rather than long-range enhancer-driven — use POSTRE's coding pathomechanism predictions instead
- The disease-relevant cell type lacks genomic data in POSTRE (e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so SHH duplication cases fail)
- The phenotype of interest is recessive — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated

## Inputs

- Structural variant file: plain text, 1 line and 7 tab-separated columns per SV (chr/breakpoint1, hg19 coordinate or comma-separated range, chr/breakpoint2, hg19 coordinate or range, SV type among Inversion/Translocation/Deletion/Duplication, comma-separated phenotype list, unique SV id)
- Control H3K27ac bigWig for the phenotype-matched cell type
- Patient H3K27ac bigWig for the phenotype-matched cell type
- Cell-type-specific TAD boundaries (hg19)
- Enhancer BED intervals for the phenotype-matched cell type

## Outputs

- enhancer_activity_scores.tsv: tab-separated table with columns gene, phenotype, enhancerActivityControl, enhancerActivityPatient, geneEnhancerScore_LOF/GOF
- Per-gene pathogenic mechanism labels distinguishing enhancer disconnection (LOF) from enhancer adoption (GOF)

## How to apply

First derive the rearranged enhancer-gene domain per SV: order breakpoints so breakpoint 1 has the smaller hg19 coordinate for strictly intrachromosomal SVs (Deletion, Inversion, Duplication), then classify each breakpoint as intra- or inter-TAD against cell-type-specific TAD boundaries. For every enhancer in the affected domains, compute mean H3K27ac signal with bigWigAverageOverBed from the control bigWig (enhancerActivityControl) and the phenotype-matched patient bigWig (enhancerActivityPatient; phenotype must be one of head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary). Aggregate enhancer-level activity changes onto candidate genes within the rearranged domain, detecting enhancer disconnection (activity loss) and enhancer adoption (activity gain). Assign geneEnhancerScore_LOF = 1 when enhancerActivityPatient_LOF is lower than enhancerActivityControl (higher enhancer activity in the wild-type/control regulatory domain), and 0 otherwise; symmetrically label GOF for activity gain. Write the per-candidate-gene table (gene, phenotype, enhancerActivityControl, enhancerActivityPatient, geneEnhancerScore_LOF/GOF) as a tab-separated file enhancer_activity_scores.tsv.

## Related tools

- **POSTRE** (Computational framework whose POSTRE_multiSV() function derives the rearranged regulatory domains and computes the geneEnhancerScore_LOF/GOF labels per candidate gene) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function invoked on a 7-column SV data frame to run the per-SV and per-gene enhancer scoring pipeline and write the results tables) — https://github.com/vicsanga/Postre
- **bigWigAverageOverBed** (Extracts mean H3K27ac signal (enhancerActivityControl / enhancerActivityPatient) per enhancer interval in the affected domains)
- **XiaoTaoWang/domaincaller** (Directionality Index TAD caller whose TAD boundaries support the intra- vs inter-TAD breakpoint classification used to build rearranged domains)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Every candidate gene row carries exactly one binary LOF or GOF geneEnhancerScore label (no gene scored both ways, no missing label)
- geneEnhancerScore_LOF = 1 exactly when enhancerActivityPatient_LOF < enhancerActivityControl; spot-check numeric columns against this rule
- Benchmark reproduction: POSTRE predicts loss of TFAP2A expression in neural crest cells through enhancer disconnection for the BOFS syndrome inversion patient (Laugsch et al., 2019)
- Output file enhancer_activity_scores.tsv is tab-separated with the expected columns gene, phenotype, enhancerActivityControl, enhancerActivityPatient, geneEnhancerScore_LOF/GOF
- Breakpoint ordering invariant: for strictly intrachromosomal SVs (deletions, inversions, duplications), breakpoint 1 must hold the smaller hg19 coordinate before intra-/inter-TAD classification

## Limitations

- Bulk H3K27ac data (e.g. whole limb buds) may not resolve small specific enhancers such as the ZRS, so POSTRE can fail to predict cases like the SHH duplication
- Prediction of long-range pathomechanisms is limited by the incomplete characterization of enhancers across human cell types
- Lack of genomic data for the appropriate developmental stages or differentiation time points can affect identification of pathologically relevant genes
- Additional enhancer-gene compatibility factors (linear distance, CpG islands, promoter DNA methylation, core promoter elements) are not yet incorporated into the scoring

## Evidence

- [methods] geneEnhancerScore_LOF assignment rule: [evidence span withheld — anchor status: not_found]
- [methods] BOFS benchmark case: [evidence span withheld — anchor status: not_found]
