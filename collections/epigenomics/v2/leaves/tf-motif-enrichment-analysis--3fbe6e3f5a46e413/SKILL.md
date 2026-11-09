---
name: tf-motif-enrichment-analysis
description: Use when you have a gene set of interest (e.g. differentially expressed
  genes) and need to discover which TF motifs are overrepresented in the regulatory
  elements EpiRegio associates with those genes in a specific cell type or tissue.
when_to_use_negative:
- When you already have TF ChIP-seq peak regions and only want to find which REMs
  and target genes they overlap — use EpiRegio's Region Query workflow instead of
  motif enrichment
- When you need to find TF binding sites within the REMs of a single gene of interest
  rather than motifs enriched across a ranked REM set — use the identifyTFBindingSites
  scenario
- When no meaningful ranking (e.g. Cell type score) separates high- from low-ranked
  sequences, since PASTAA compares high-ranked against low-ranked input sequences
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3176
tools:
- name: EpiRegio
  role: Gene Query to identify REMs associated with the genes of interest, filtered
    by cell type/tissue, with results downloaded as CSV
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: bedtools
  role: getfasta to extract REM DNA sequences from the genome assembly into REMs.fa
- name: PASTAA
  role: Performs the TF motif enrichment analysis on REM sequences ranked by Cell
    type score, reporting Benjamini–Hochberg adjusted p-values
  repo: https://github.com/TeamRegio/ApplicationScenarioExamples
- name: TRAP
  role: Provides TF affinities normalized by TF binding motif length used within PASTAA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:9ae01eb4b86d176fa9d553904233b7c4d60f79e19d8eb592f590c613798321fe
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
  - http://edamontology.org/topic_3176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# tf-motif-enrichment-analysis

## Summary

Identifies transcription factors whose binding motifs are significantly enriched in REM sequences of differentially expressed genes, ranked by EpiRegio's Cell type score, using PASTAA with a Benjamini–Hochberg adjusted p-value cutoff of 0.05.

## When to use

Use when you have a gene set of interest (e.g. differentially expressed genes) and need to discover which TF motifs are overrepresented in the regulatory elements EpiRegio associates with those genes in a specific cell type or tissue.

## When NOT to use

- When you already have TF ChIP-seq peak regions and only want to find which REMs and target genes they overlap — use EpiRegio's Region Query workflow instead of motif enrichment
- When you need to find TF binding sites within the REMs of a single gene of interest rather than motifs enriched across a ranked REM set — use the identifyTFBindingSites scenario
- When no meaningful ranking (e.g. Cell type score) separates high- from low-ranked sequences, since PASTAA compares high-ranked against low-ranked input sequences

## Inputs

- Differentially expressed gene set (e.g. GeneSet.txt from TeamRegio/ApplicationScenarioExamples)
- EpiRegio Gene Query REM table downloaded as CSV, filtered for cell type/tissue (e.g. 'heart')
- Human genome assembly in FASTA format from UCSC
- JASPAR 2020 human motif file in TRANSFAC format (JASPAR2020_HUMAN_transfac.txt)

## Outputs

- <outputDir>/PASTAA_result.txt listing TF motifs with Benjamini–Hochberg adjusted p-value <= 0.05
- REMs.fa containing extracted REM DNA sequences
- BED file of REM coordinates (chromosome, start, end)

## How to apply

Run EpiRegio's Gene Query on the gene set (e.g. GeneSet.txt), filtering for the relevant cell type/tissue such as 'heart', and download the REM table as CSV. Convert the REM coordinates to a BED file and extract their DNA sequences with 'bedtools getfasta' against a human genome FASTA from UCSC into REMs.fa. Compile PASTAA from the ApplicationScenarioExamples repository (with TRAP affinities normalized by TF binding motif length) via 'make' in its src folder, then run the provided identifyEnrichedTFs/workflow.sh with a JASPAR 2020 human motif file in TRANSFAC format (JASPAR2020_HUMAN_transfac.txt), the genome FASTA, the REM CSV, and a Benjamini–Hochberg adjusted p-value cutoff of 0.05. PASTAA identifies TF binding motifs significantly enriched in high-ranked input sequences compared to low-ranked sequences, so ranking REMs by the Cell type score is the key decision point that supplies the required cell-type-specific signal.

## Related tools

- **EpiRegio** (Gene Query to identify REMs associated with the genes of interest, filtered by cell type/tissue, with results downloaded as CSV) — https://github.com/TeamRegio/EpiRegioDB
- **bedtools** (getfasta to extract REM DNA sequences from the genome assembly into REMs.fa)
- **PASTAA** (Performs the TF motif enrichment analysis on REM sequences ranked by Cell type score, reporting Benjamini–Hochberg adjusted p-values) — https://github.com/TeamRegio/ApplicationScenarioExamples
- **TRAP** (Provides TF affinities normalized by TF binding motif length used within PASTAA)

## Examples

```
bash identifyEnrichedTFs/workflow.sh JASPAR2020_HUMAN_transfac.txt hg38.fa REMs.csv 0.05
```

## Evaluation signals

- <outputDir>/PASTAA_result.txt exists and contains only TF motifs whose Benjamini–Hochberg adjusted p-value is <= 0.05
- Recovery of the reported 230 enriched TFs, including EndoMT-relevant SP1, NFKB1, and SMAD3, among the significant motifs
- REMs.fa contains one sequence per REM matching the BED coordinates extracted from the current UCSC human genome assembly
- REM table downloaded from EpiRegio Gene Query reflects the applied cell type/tissue filter (e.g. 'heart')
- PASTAA was compiled with TRAP affinities normalized by TF binding motif length (make in src folder completed without error)

## Limitations

- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score, is a simplification of actual gene regulation by REMs
- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, etc.)
- Results depend on the supplied genome assembly and JASPAR 2020 motif set; a current human genome FASTA from UCSC is required

## Evidence

- [discussion] we used PASTAA, which identifies TF binding motifs that are significantly enriched in high ranked input sequences compared to low ranked sequences: "we used PASTAA, which identifies TF binding motifs that are significantly enriched in high ranked input sequences compared to low ranked sequences"
- [discussion] In order to rank the obtained REMs we sorted them according to their Cell type score: "In order to rank the obtained REMs we sorted them according to their Cell type score"
- [methods] we determine the DNA-sequence of the identified REMs using *bedtools*: "we determine the DNA-sequence of the identified REMs using *bedtools*"
- [methods] If the Benjamini-Hochberg adjusted p-value from PASTAA is smaller than or equal to the parameter *<pvalue>* the motif is assumed to be significant enriched.: "If the Benjamini-Hochberg adjusted p-value from PASTAA is smaller than or equal to the parameter *<pvalue>* the motif is assumed to be significant enriched."
- [readme] For application scenario "How to use EpiRegio to identify enriched TFs of a set of genes of interest" the examples and the workflow script can be found in the folder identifyEnrichedTFs.: [evidence span withheld — anchor status: external_doc]
