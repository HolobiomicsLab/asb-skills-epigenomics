---
name: chi-square-contingency-analysis
description: Use when comparing categorical occurrence counts (e.g., motif families
  from Genomatix MatBase) between test and control sequence sets to quantify which
  category is most enriched, such as STAT motif enrichment in ChIP-Seq peaks versus
  ChIP-chip regions.
when_to_use_negative:
- Peak calling or read-enriched region definition is still needed — use HPeak (HMM
  with GP/ZIP emissions) first; chi-square analysis only evaluates the resulting sequences
- The motifs of interest are not represented in a motif matrix database (the authors
  note many stored motifs may be inaccurate and novel motifs may not exist in databases)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: MatInspector
  role: Scans test and control sequences for motif family matches under default settings,
    producing the occurrence counts feeding the chi-square tables
- name: Genomatix suite
  role: Software environment in which MatInspector motif scans against MatBase matrices
    are run
- name: HPeak
  role: Defines the ChIP-Seq peak regions and generates the FASTA sequence files whose
    motif content is compared against ChIP-chip regions
  repo: http://www.sph.umich.edu/csg/qin/HPeak
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/1471-2105-11-369
    title: 'HPeak: an HMM-based algorithm for defining read-enriched regions in ChIP-Seq
      data'
  source_content_hash: sha256:617bc660237f0401ff6a89d3788a7c6e7e0b2910bbdf1880bf44df3da69e4a9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Genomatix suite
  - HPeak
  - MatInspector
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# chi-square-contingency-analysis

## Summary

A family-level motif enrichment comparison that tabulates MatInspector motif family occurrence counts in test versus control sequences and applies chi-square statistics to test each motif family's enrichment, as used to show STAT family enrichment in STAT1 ChIP-Seq peaks exceeds that in ChIP-chip regions.

## When to use

Use when comparing categorical occurrence counts (e.g., motif families from Genomatix MatBase) between test and control sequence sets to quantify which category is most enriched, such as STAT motif enrichment in ChIP-Seq peaks versus ChIP-chip regions.

## When NOT to use

- Peak calling or read-enriched region definition is still needed — use HPeak (HMM with GP/ZIP emissions) first; chi-square analysis only evaluates the resulting sequences
- The motifs of interest are not represented in a motif matrix database (the authors note many stored motifs may be inaccurate and novel motifs may not exist in databases)

## Inputs

- Genomic FASTA sequences underlying ChIP-Seq peaks (HPeak -seq output)
- Genomic FASTA sequences underlying ChIP-chip regions (Euskirchen et al., 3,701 regions, chromosomes 20/21/22/X/Y, FDR 0.05)
- Matched control genomic sequences
- Motif position frequency matrices (Genomatix MatBase 7.0, 153 vertebrate motif families)

## Outputs

- Motif family occurrence count tables for test and control sequences
- Per-family chi-square enrichment statistics (negative log transformed p-values)
- Rank-binned motif enrichment profiles per platform
- motif_enrichment_results.csv

## How to apply

Extract genomic sequences underlying called peaks (e.g., STAT1 ChIP-Seq peaks from HPeak and the 3,701 STAT1 ChIP-chip regions from Euskirchen et al. on chromosomes 20, 21, 22, X, Y at FDR 0.05) plus matched control sequences. Scan all test and control sequences with MatInspector in the Genomatix suite using default settings against all 153 families of vertebrate motif matrices in Genomatix MatBase 7.0. Tabulate motif family occurrence counts in test versus control sequences and compute a chi-square statistic per motif family quantifying enrichment. Compare the STAT family chi-square enrichment to the second-highest family per platform; correct application reproduces the Figure 1A/B result where the negative log transformed p-value for the STAT family in ChIP-Seq peaks is about twice that of the runner-up family. Optionally stratify peaks by rank bins to examine rank dependence of enrichment (Figure 1C trend of steeper decay in ChIP-Seq than ChIP-chip), then write family-level statistics to a CSV.

## Related tools

- **MatInspector** (Scans test and control sequences for motif family matches under default settings, producing the occurrence counts feeding the chi-square tables)
- **Genomatix suite** (Software environment in which MatInspector motif scans against MatBase matrices are run)
- **HPeak** (Defines the ChIP-Seq peak regions and generates the FASTA sequence files whose motif content is compared against ChIP-chip regions) — http://www.sph.umich.edu/csg/qin/HPeak

## Examples

```
perl /compbio/software/HPeak3/HPeak.pl -sp HUMAN -format BED -t TREATMENT.inp -c CONTROL.inp -n OUTPREFIX -r 36 -ann -wig -seq -interfiles  # HPeak -seq output supplies the FASTA sequences later counted per motif family for chi-square tests
```

## Evaluation signals

- In ChIP-Seq peak sequences, the STAT motif family's chi-square enrichment (negative log transformed p-value) should be about twice that of the second-highest motif family
- STAT family enrichment should rank first among all 153 vertebrate motif families in both ChIP-Seq and ChIP-chip comparisons
- Rank-binned enrichment should show a steeper decay with peak rank in ChIP-Seq than in ChIP-chip (Figure 1C trend)
- Output CSV should contain one row per motif family with test/control counts and a chi-square statistic for both platforms

## Limitations

- Many motif patterns stored in databases may be inaccurate, and novel motifs may not exist in motif databases; the effect of inaccurate motif patterns on the comparison remains to be seen
- Little information on true positive and true negative loci makes performance evaluation on experimental ChIP-Seq data challenging, so results should be interpreted with caution
- ChIP-chip comparisons are limited to regions assayed on the platform (Euskirchen et al. covered chromosomes 20, 21, 22, X, Y at FDR 0.05), which restricts direct count comparability

## Evidence

- [discussion] Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite: "Motif scan of the test and the control sequences was then performed using MatInspector [52] in the Genomatix suite"
- [discussion] all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database: "all 153 families of vertebrate motif matrices found in the Genomatix MatBase 7.0 database"
- [discussion] the ChIP-Seq technique has a clear advantage over the ChIP-chip technique in terms of enriching for an expected motif under the predicted peaks: "the ChIP-Seq technique has a clear advantage over the ChIP-chip technique in terms of enriching for an expected motif under the predicted peaks"
- [readme] Accurate peak calling with detailed information including summit location and p-value: "Accurate peak calling with detailed information including summit location and p-value"
