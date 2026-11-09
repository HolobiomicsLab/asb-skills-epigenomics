---
name: cell-type-specific-regulatory-landscape-integration
description: Use when interpreting patient structural variants (inversions, deletions,
  duplications, translocations) whose pathogenic effect is suspected to be non-coding/long-range,
  and cell type-specific enhancer and TAD data are needed to nominate disease genes.
when_to_use_negative:
- The SV is a coding variant or clearly disrupts the coding sequence of a known gene
  — POSTRE's long-range enhancer scoring is unnecessary and the intronic/truncating
  filter handles this differently
- The phenotype of interest falls outside the 7 supported contexts (head_neck, limbs,
  neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary)
  or lacks genomic data for the appropriate developmental stage
- Regulatory landscapes are only available as bulk whole-tissue data that cannot resolve
  focal enhancers (e.g. bulk whole limb bud data cannot identify the ZRS enhancer,
  so the SHH duplication case is mispredicted)
tools:
- name: POSTRE
  role: Core tool that derives rearranged enhancer-gene domains from cell-type-specific
    TADs and enhancers and predicts SV pathomechanisms
  repo: https://github.com/vicsanga/Postre
- name: POSTRE_multiSV()
  role: R function entry point for batch analysis of multiple SVs against the regulatory
    landscape; source POSTRE_multiSV.R then call with SVs data frame and Postre app
    folder path
  repo: https://github.com/vicsanga/Postre
- name: bigWigAverageOverBed
  role: Extracts mean H3K27ac signal (enhancerIndividualActivity) per enhancer interval
    from control and patient bigWig files
- name: domaincaller
  role: Directionality-Index TAD caller (Dixon et al. 2012) referenced for deriving
    TAD boundaries from Hi-C data
  repo: https://github.com/XiaoTaoWang/domaincaller
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad225
    title: 'POSTRE: a tool to predict the pathological effects of human structural
      variants'
  source_content_hash: sha256:a1ac451f97a792982535f26d8e44b1c8ce7343601ca81d06b8a18f253eedf754
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9259259259259259
n_claims: 27
---

# cell-type-specific-regulatory-landscape-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates cell type/tissue-specific enhancer (H3K27ac) and TAD regulatory landscapes into structural variant interpretation via POSTRE, so that long-range (enhancer-driven) pathomechanisms such as enhancer disconnection or adoption can be predicted for congenital disorder SVs.

## When to use

Use when interpreting patient structural variants (inversions, deletions, duplications, translocations) whose pathogenic effect is suspected to be non-coding/long-range, and cell type-specific enhancer and TAD data are needed to nominate disease genes.

## When NOT to use

- The SV is a coding variant or clearly disrupts the coding sequence of a known gene — POSTRE's long-range enhancer scoring is unnecessary and the intronic/truncating filter handles this differently
- The phenotype of interest falls outside the 7 supported contexts (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system, pituitary) or lacks genomic data for the appropriate developmental stage
- Regulatory landscapes are only available as bulk whole-tissue data that cannot resolve focal enhancers (e.g. bulk whole limb bud data cannot identify the ZRS enhancer, so the SHH duplication case is mispredicted)

## Inputs

- Plain text SV file: 1 line per SV with 7 tab-separated columns (chr1, hg19 coordinates1, chr2, coordinates2, SV type, phenotype list, SV id); ranges as comma-separated values when not base-pair resolution
- Cell-type-specific enhancer and TAD boundary data bundled with POSTRE for the 7 supported phenotypes
- Control and patient phenotype-matched bigWig files for H3K27ac signal extraction

## Outputs

- enhancer_activity_scores.tsv: per-candidate-gene table with gene, phenotype, enhancerActivityControl, enhancerActivityPatient and binary geneEnhancerScore_LOF/GOF
- Results per SV and phenotype table with a pathogenic prediction for each SV/phenotype
- Results per gene and phenotype table aggregating pathogenic predictions per gene, phenotype and pathogenic mechanism (coding, long-range)

## How to apply

Format each SV as one line with 7 tab-separated columns (chromosome 1, hg19 coordinates 1, chromosome 2, coordinates 2, SV type among Inversion/Translocation/Deletion/Duplication, comma-separated phenotype list, unique SV id); for strictly intrachromosomal SVs order breakpoints so breakpoint 1 has the smaller genomic coordinate. Restrict phenotypes to the supported contexts (head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary), since POSTRE's regulatory landscape is phenotype-matched. POSTRE then classifies each breakpoint as intra- or inter-TAD relative to cell-type-specific TAD boundaries to derive the rearranged enhancer-gene domain, computes mean H3K27ac signal per enhancer with bigWigAverageOverBed from control and patient bigWig files, and aggregates enhancer-level activity changes onto candidate genes to call enhancer disconnection (activity loss) or enhancer adoption (activity gain). Each candidate gene receives a binary geneEnhancerScore_LOF or geneEnhancerScore_GOF label (LOF = 1 when enhancerActivityPatient is lower than enhancerActivityControl). The rationale is that SV pathomechanisms can only be predicted and validated if cell type-specific regulatory landscapes (enhancers and TADs) are considered, which POSTRE automates in a Shiny/R (>=3.5.0) framework.

## Related tools

- **POSTRE** (Core tool that derives rearranged enhancer-gene domains from cell-type-specific TADs and enhancers and predicts SV pathomechanisms) — https://github.com/vicsanga/Postre
- **POSTRE_multiSV()** (R function entry point for batch analysis of multiple SVs against the regulatory landscape; source POSTRE_multiSV.R then call with SVs data frame and Postre app folder path) — https://github.com/vicsanga/Postre
- **bigWigAverageOverBed** (Extracts mean H3K27ac signal (enhancerIndividualActivity) per enhancer interval from control and patient bigWig files)
- **domaincaller** (Directionality-Index TAD caller (Dixon et al. 2012) referenced for deriving TAD boundaries from Hi-C data) — https://github.com/XiaoTaoWang/domaincaller

## Examples

```
source("/PathTo/Postre_app/POSTRE_multiSV.R"); POSTRE_multiSV(SVs, pathTo_Postre_app_Folder)
```

## Evaluation signals

- Reproduce the benchmark: POSTRE predicts loss of TFAP2A expression in neural crest cells through enhancer disconnection for the BOFS syndrome inversion patient (Laugsch et al., 2019)
- Every gene row in enhancer_activity_scores.tsv carries exactly one binary geneEnhancerScore_LOF or geneEnhancerScore_GOF label
- geneEnhancerScore_LOF = 1 exactly when enhancerActivityPatient is lower than enhancerActivityControl for the candidate gene
- SV file validates as 1 line / 7 tab-separated columns per SV, with SV type in {Inversion, Translocation, Deletion, Duplication} and phenotype restricted to the 7 supported options
- Two output tables are produced: one per SV/phenotype with a pathogenic prediction, and one aggregated per gene/phenotype/mechanism (coding, long-range)

## Limitations

- Bulk genomic data may lack resolution for focal enhancers (e.g. whole limb bud data cannot identify the ZRS enhancer, so POSTRE fails to predict the SHH duplication case)
- Incomplete characterization of enhancers across human cell types limits prediction of long-range pathomechanisms
- POSTRE is oriented towards dominant disorders; its capacity for recessive phenotypes has not been specifically evaluated
- Lack of genomic data for the appropriate developmental stages or differentiation time points can affect identification of pathologically relevant genes
- Enhancer-gene compatibility factors (linear distance, CpG islands, promoter methylation, core promoter elements) are not yet incorporated into POSTRE's scoring

## Evidence

- [intro] the exact pathomechanisms can only be predicted and experimentally validated if cell type specific regulatory landscapes (i.e. enhancers and TADs) are considered: "the exact pathomechanisms can only be predicted and experimentally validated if cell type specific regulatory landscapes (i.e. enhancers and TADs) are considered"
- [readme] POSTRE successfully predicts the loss of TFAP2A expression in neural crest cells through an enhancer disconnection mechanism.: "POSTRE successfully predicts the loss of TFAP2A expression in neural crest cells through an enhancer disconnection mechanism."
- [readme] Current options are: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary.: "Current options are: head_neck, limbs, neurodevelopmental, cardiovascular, vision_eye, liver_biliary_system or pituitary."
- [abstract] By considering disease-relevant cellular contexts, POSTRE identifies SVs with either coding or long-range pathological consequences with high specificity and sensitivity.: "By considering disease-relevant cellular contexts, POSTRE identifies SVs with either coding or long-range pathological consequences with high specificity and sensitivity."
- [methods] do a source of the POSTRE_multiSV.R script (i.e. source("/PathTo/Postre_app/POSTRE_multiSV.R")): "do a source of the POSTRE_multiSV.R script (i.e. source("/PathTo/Postre_app/POSTRE_multiSV.R"))"
