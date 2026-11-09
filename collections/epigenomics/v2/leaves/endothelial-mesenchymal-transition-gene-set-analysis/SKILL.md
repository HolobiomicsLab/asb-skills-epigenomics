---
name: endothelial-mesenchymal-transition-gene-set-analysis
description: Use when a set of differentially expressed genes (e.g. during EndoMT)
  must be linked to regulatory elements and screened for enriched TF binding motifs
  in a cell type-specific manner.
when_to_use_negative:
- The goal is to identify a TF's target genes from ChIP-seq peaks rather than to find
  TFs enriched in REMs of a gene set — use EpiRegio's Region Query workflow instead.
- The input is already a set of REM sequences with a ranking, so no gene-to-REM association
  step is needed — run PASTAA directly rather than via the Gene Query pipeline.
- The task requires mapping TF binding sites within REMs of a single gene of interest
  — use the identifyTFBindingSites scenario from the ApplicationScenarioExamples repository
  instead.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3179
- http://edamontology.org/topic_0203
tools:
- name: EpiRegio
  role: Gene Query web server used to identify REMs associated with the EndoMT differentially
    expressed gene set, filtered for cell type/tissue 'heart', returning the REM table
    with Cell type score for ranking
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: bedtools
  role: bedtools getfasta extracts the DNA sequences of the identified REMs from a
    human genome FASTA into REMs.fa for motif enrichment
- name: PASTAA
  role: Performs TF motif enrichment analysis on REM sequences ranked by Cell type
    score, with TRAP affinities normalized by TF binding motif length and Benjamini–Hochberg
    FDR cutoff 0.05
  repo: https://github.com/TeamRegio/ApplicationScenarioExamples/
- name: TRAP
  role: Computes TF affinity predictions whose affinities are normalized by TF binding
    motif length inside the PASTAA compilation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:c26732877382131dc2fdd7ba422da7beaed7a90ecc537ae87efac59b070106e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - PASTAA
  - TRAP
  - bedtools
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3179
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# endothelial-mesenchymal-transition-gene-set-analysis

## Summary

Identifies transcription factors whose binding motifs are enriched in REMs of genes differentially expressed during TGF-β-induced EndoMT in HUVECs, by running EpiRegio's Gene Query (filtered for 'heart'), ranking REMs by Cell type score, and applying PASTAA motif enrichment with Benjamini–Hochberg FDR ≤ 0.05.

## When to use

Use when a set of differentially expressed genes (e.g. during EndoMT) must be linked to regulatory elements and screened for enriched TF binding motifs in a cell type-specific manner.

## When NOT to use

- The goal is to identify a TF's target genes from ChIP-seq peaks rather than to find TFs enriched in REMs of a gene set — use EpiRegio's Region Query workflow instead.
- The input is already a set of REM sequences with a ranking, so no gene-to-REM association step is needed — run PASTAA directly rather than via the Gene Query pipeline.
- The task requires mapping TF binding sites within REMs of a single gene of interest — use the identifyTFBindingSites scenario from the ApplicationScenarioExamples repository instead.

## Inputs

- GeneSet.txt — set of genes differentially expressed during TGF-β-induced EndoMT in HUVECs (from TeamRegio/ApplicationScenarioExamples)
- Human genome assembly in FASTA format (from UCSC)
- JASPAR 2020 human motif file in TRANSFAC format (JASPAR2020_HUMAN_transfac.txt)
- Cell type/tissue filter term ('heart') for EpiRegio's Gene Query

## Outputs

- EpiRegio REM table (CSV) listing REMs associated with the input genes, including Cell type score
- REMs.bed — BED file of REM coordinates (chromosome, start, end)
- REMs.fa — REM DNA sequences extracted with bedtools getfasta
- PASTAA_result.txt — list of TF motifs with Benjamini–Hochberg adjusted p-values; 230 TFs significant at FDR ≤ 0.05, including SP1, NFKB1, and SMAD3

## How to apply

Run EpiRegio's Gene Query on the differentially expressed gene set (GeneSet.txt from TeamRegio/ApplicationScenarioExamples), entering 'heart' in the 'Filter for cell types/tissues' field, and download the REM table as CSV. Convert the CSV to a BED file (chromosome, start, end) and extract REM DNA sequences with 'bedtools getfasta' against a current human genome FASTA from UCSC, writing to REMs.fa. Clone the ApplicationScenarioExamples repository and compile PASTAA (with TRAP affinities normalized by TF binding motif length) via 'make' in its src folder. Run the 'identifyEnrichedTFs/workflow.sh' script with the JASPAR 2020 human motif file in TRANSFAC format (JASPAR2020_HUMAN_transfac.txt), the human genome FASTA, the REM CSV, and a Benjamini–Hochberg adjusted p-value cutoff of 0.05. The rationale is that PASTAA identifies TF binding motifs significantly enriched in high-ranked versus low-ranked input sequences, and EpiRegio's Cell type score (derived from the STITCHIT linear model of chromatin accessibility vs gene expression) provides the cell type-specific ranking.

## Related tools

- **EpiRegio** (Gene Query web server used to identify REMs associated with the EndoMT differentially expressed gene set, filtered for cell type/tissue 'heart', returning the REM table with Cell type score for ranking) — https://github.com/TeamRegio/EpiRegioDB
- **bedtools** (bedtools getfasta extracts the DNA sequences of the identified REMs from a human genome FASTA into REMs.fa for motif enrichment)
- **PASTAA** (Performs TF motif enrichment analysis on REM sequences ranked by Cell type score, with TRAP affinities normalized by TF binding motif length and Benjamini–Hochberg FDR cutoff 0.05) — https://github.com/TeamRegio/ApplicationScenarioExamples/
- **TRAP** (Computes TF affinity predictions whose affinities are normalized by TF binding motif length inside the PASTAA compilation)

## Examples

```
bash identifyEnrichedTFs/workflow.sh JASPAR2020_HUMAN_transfac.txt hg38.fa GeneQueryResult.csv 0.05 <outputDir>
```

## Evaluation signals

- The output file <outputDir>/PASTAA_result.txt exists and contains only TF motifs whose Benjamini–Hochberg adjusted p-value is <= 0.05.
- The reported enriched TFs — 230 TFs including SP1, NFKB1, and SMAD3 — are recovered among the significant motifs.
- REMs.fa contains one FASTA entry per REM row in the downloaded CSV, with sequences matching the human genome assembly coordinates in REMs.bed.
- The Gene Query result was filtered for cell type/tissue 'heart', so all REMs in the CSV carry Cell type score values for that tissue.
- PASTAA was compiled with TRAP affinities normalized by TF binding motif length (verified by the 'make' build in the src folder of ApplicationScenarioExamples).

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs.
- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, etc.), so exact reproducibility may vary across releases.
- The Cell type score threshold is configurable; results depend on the chosen thresholds for Cell type score and Cell type DNase1 signal of the REMs.

## Evidence

- [methods] Gene Query to identify REMs associated to genes of interest filtered by cell type/tissue: "Use EpiRegio's `Gene Query <https://epiregiodb.readthedocs.io/en/latest/UseCases.html#query-guide>`_ to identify the REMs associated to the genes of interest"
- [methods] Extract REM DNA sequences with bedtools getfasta: "we determine the DNA-sequence of the identified REMs using *bedtools*"
- [methods] Run PASTAA motif enrichment analysis on REM sequences: "run *PASTAA* to perform the motif enrichment analysis"
- [discussion] PASTAA identifies enriched TF motifs in ranked sequences: "we used PASTAA, which identifies TF binding motifs that are significantly enriched in high ranked input sequences compared to low ranked sequences"
- [discussion] STITCHIT Cell type score is a simplification of gene regulation: "the linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification to the actual regulation of genes by their REMs"
- [methods] Filter for cell types/tissues in Gene Query: "Enter *heart* in the field *Filter for cell types/tissues*"
- [readme] ApplicationScenarioExamples contains the identifyEnrichedTFs workflow: [evidence span withheld — anchor status: external_doc]
