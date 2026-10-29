---
name: tad-boundary-rearrangement-classification
description: Use when a 7-column SV record (breakpoints in hg19, SV type Inversion/Translocation/Deletion/Duplication,
  phenotype list, SV id) needs its breakpoints classified against cell-type-specific
  TAD boundaries to build the rearranged enhancer–gene domain.
when_to_use_negative:
- The phenotype of interest is a recessive disorder — POSTRE is oriented towards dominant
  disorders and its capacity for recessive phenotypes has not been specifically evaluated
- Only bulk tissue data is available where the relevant cell-type-specific enhancer
  is unidentifiable (e.g. POSTRE fails on the SHH duplication case because whole limb
  bud data cannot identify the ZRS enhancer)
- Genomic data for the appropriate developmental stage or differentiation time point
  is lacking, so the TAD/enhancer landscape does not reflect the disease-relevant
  context
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: POSTRE
  role: Shiny/R tool that applies the TAD-based rearranged domain rules to classify
    SVs and predict pathomechanisms cell-type-specifically
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function called with a 7-column SV data frame and the Postre_app folder
    path to run the multi-SV analysis that includes breakpoint TAD classification
  repo: https://github.com/vicsanga/Postre
- name: domaincaller
  role: Directionality Index HMM TAD caller whose cell-type-specific TAD boundaries
    underpin the intra-/inter-TAD breakpoint classification
  repo: XiaoTaoWang/domaincaller
- name: bigWigAverageOverBed
  role: Extracts mean H3K27ac signal per enhancer interval within the rearranged domains
    from control and patient bigWig files
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:924305adb50916b54fa9dad1f374a68cd91eec957c2955e95a44ecd740356b02
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
  - domaincaller
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# tad-boundary-rearrangement-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Derives the rearranged enhancer–gene regulatory domain produced by a structural variant by ordering breakpoints and classifying each breakpoint as intra- or inter-TAD against cell-type-specific TAD boundaries, enabling POSTRE's enhancer disconnection/adoption predictions.

## When to use

Use when a 7-column SV record (breakpoints in hg19, SV type Inversion/Translocation/Deletion/Duplication, phenotype list, SV id) needs its breakpoints classified against cell-type-specific TAD boundaries to build the rearranged enhancer–gene domain.

## When NOT to use

- The phenotype of interest is a recessive disorder — POSTRE is oriented towards dominant disorders and its capacity for recessive phenotypes has not been specifically evaluated
- Only bulk tissue data is available where the relevant cell-type-specific enhancer is unidentifiable (e.g. POSTRE fails on the SHH duplication case because whole limb bud data cannot identify the ZRS enhancer)
- Genomic data for the appropriate developmental stage or differentiation time point is lacking, so the TAD/enhancer landscape does not reflect the disease-relevant context

## Inputs

- Plain text SV file with 1 line and 7 tab-separated columns per SV: chr/breakpoint1, hg19 coordinates (or comma-separated range), chr/breakpoint2, coordinates (or range), SV type (Inversion/Translocation/Deletion/Duplication), comma-separated phenotype list, unique SV id
- Cell-type-specific TAD boundaries for the selected phenotype context
- Phenotype-matched control and patient H3K27ac bigWig files

## Outputs

- Rearranged enhancer–gene regulatory domain assignment per SV breakpoint (intra- vs inter-TAD classification)
- Per-candidate-gene table enhancer_activity_scores.tsv with gene, phenotype, enhancerActivityControl, enhancerActivityPatient, and one binary geneEnhancerScore_LOF/GOF label per gene
- Aggregated per gene/phenotype/mechanism pathogenic predictions (coding, long-range)

## How to apply

Parse each SV line (chr/breakpoint1, hg19 coordinate or comma-separated range, chr/breakpoint2, coordinate/range, SV type, comma-separated phenotypes, SV id). For strictly intrachromosomal SVs (deletions, inversions, duplications) order breakpoints so breakpoint 1 holds the smaller genomic coordinate; for translocations ordering does not matter. Classify each breakpoint as intra- or inter-TAD relative to the TAD boundaries of the phenotype-matched cell type (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary), which determines the rearranged enhancer–gene domain. This domain is the substrate for computing enhancerActivityControl and enhancerActivityPatient (mean H3K27ac via bigWigAverageOverBed) and for assigning binary geneEnhancerScore_LOF/GOF labels. The rationale is that pathomechanisms such as enhancer disconnection and enhancer adoption are only interpretable in a cell-type-specific regulatory landscape of enhancers and TADs.

## Related tools

- **POSTRE** (Shiny/R tool that applies the TAD-based rearranged domain rules to classify SVs and predict pathomechanisms cell-type-specifically) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function called with a 7-column SV data frame and the Postre_app folder path to run the multi-SV analysis that includes breakpoint TAD classification) — https://github.com/vicsanga/Postre
- **domaincaller** (Directionality Index HMM TAD caller whose cell-type-specific TAD boundaries underpin the intra-/inter-TAD breakpoint classification) — XiaoTaoWang/domaincaller
- **bigWigAverageOverBed** (Extracts mean H3K27ac signal per enhancer interval within the rearranged domains from control and patient bigWig files)

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- For strictly intrachromosomal SVs, breakpoint 1 must carry the smaller genomic coordinate and breakpoint 2 the larger one
- Benchmark: POSTRE must predict loss of TFAP2A expression in neural crest cells through enhancer disconnection for the BOFS syndrome inversion patient (Laugsch et al., 2019)
- Every gene row in the output carries exactly one binary geneEnhancerScore_LOF or geneEnhancerScore_GOF label
- geneEnhancerScore_LOF = 1 when enhancerActivityPatient_LOF is lower than enhancerActivityControl, and 0 otherwise
- Output is a tab-separated enhancer_activity_scores.tsv with columns gene, phenotype, enhancerActivityControl, enhancerActivityPatient, geneEnhancerScore_LOF/GOF

## Limitations

- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms
- Additional enhancer–gene compatibility factors (linear distance, CpG islands, promoter DNA methylation, core promoter elements) are not yet incorporated into POSTRE's scoring
- POSTRE is oriented towards dominant disorders; recessive-phenotype capacity has not been specifically evaluated

## Evidence
