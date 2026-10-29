---
name: regulatory-element-sequence-extraction
description: Use when an EpiRegio Gene Query result table (CSV of REMs linked to genes
  of interest, filtered for a cell type/tissue such as 'heart') must be turned into
  DNA sequences in FASTA format to feed a motif enrichment tool.
when_to_use_negative:
- The sequences are already available in FASTA form and only motif enrichment is needed
  — skip extraction and feed the FASTA directly to PASTAA
- The task is to identify a TF's target genes from ChIP-seq peaks (Region Query of
  ChIP-seq peaks against REMs with 50% overlap), which does not require sequence extraction
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0102
tools:
- name: EpiRegio
  role: Web server that supplies the REMs and their coordinates via the Gene Query,
    whose downloaded CSV is converted to BED for sequence extraction
  repo: https://github.com/TeamRegio/EpiRegioDB
- name: bedtools
  role: 'Performs the actual sequence extraction: ''bedtools getfasta'' reads the
    BED intervals and a genome FASTA and writes REMs.fa'
- name: PASTAA
  role: Downstream consumer of REMs.fa in the identifyEnrichedTFs workflow, identifying
    TF binding motifs enriched in high ranked input sequences
  repo: https://github.com/TeamRegio/ApplicationScenarioExamples
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkaa382
    title: 'EpiRegio: analysis and retrieval of regulatory elements linked to genes'
  source_content_hash: sha256:9643e7bbae17954450889f97c1f83ca0e98a23788320456c3326cf7d0c67eca6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TeamRegio/EpiRegioDB
  tools:
  - EpiRegio
  - PASTAA
  - TRAP
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# regulatory-element-sequence-extraction

## Summary

Extract the DNA sequences of REMs identified by EpiRegio's Gene Query by converting the downloaded result CSV to a BED file (chromosome, start, end) and running bedtools getfasta against a current human genome FASTA, producing REMs.fa for downstream motif enrichment (e.g. PASTAA).

## When to use

Use when an EpiRegio Gene Query result table (CSV of REMs linked to genes of interest, filtered for a cell type/tissue such as 'heart') must be turned into DNA sequences in FASTA format to feed a motif enrichment tool.

## When NOT to use

- The sequences are already available in FASTA form and only motif enrichment is needed — skip extraction and feed the FASTA directly to PASTAA
- The task is to identify a TF's target genes from ChIP-seq peaks (Region Query of ChIP-seq peaks against REMs with 50% overlap), which does not require sequence extraction

## Inputs

- EpiRegio Gene Query result table (CSV) of REMs associated to genes of interest, filtered for a cell type/tissue
- Human genome reference sequence in FASTA format (e.g. downloaded from UCSC)
- BED file with REM coordinates (chromosome, start, end) derived from the CSV

## Outputs

- BED file of REM genomic intervals
- REMs.fa — FASTA file of REM DNA sequences for motif enrichment analysis

## How to apply

After running EpiRegio's Gene Query on the differentially expressed gene set and filtering for the relevant cell type/tissue, download the REM table via the CSV button. Convert the CSV to a BED file using its chromosome, start, and end columns, since motif tools consume genomic intervals rather than tables. Run 'bedtools getfasta' with a current human genome assembly in FASTA format downloaded from UCSC, writing the extracted sequences to REMs.fa; these ranked sequences are the direct input for PASTAA in the identifyEnrichedTFs workflow of the ApplicationScenarioExamples repository, where motifs with Benjamini–Hochberg adjusted p-value ≤ 0.05 are called enriched.

## Related tools

- **EpiRegio** (Web server that supplies the REMs and their coordinates via the Gene Query, whose downloaded CSV is converted to BED for sequence extraction) — https://github.com/TeamRegio/EpiRegioDB
- **bedtools** (Performs the actual sequence extraction: 'bedtools getfasta' reads the BED intervals and a genome FASTA and writes REMs.fa)
- **PASTAA** (Downstream consumer of REMs.fa in the identifyEnrichedTFs workflow, identifying TF binding motifs enriched in high ranked input sequences) — https://github.com/TeamRegio/ApplicationScenarioExamples

## Examples

```
bedtools getfasta -fi hg38.fa -bed REMs.bed -fo REMs.fa
```

## Evaluation signals

- The output REMs.fa exists and contains one FASTA record per REM in the BED/CSV input
- Each FASTA header/record corresponds to a REM interval with chromosome, start, end matching the BED file
- Sequence lengths in REMs.fa equal the end-minus-start lengths of the corresponding BED intervals
- The extracted sequences are usable downstream: running the identifyEnrichedTFs workflow on them yields a PASTAA_result.txt containing only motifs with Benjamini–Hochberg adjusted p-value ≤ 0.05
- The REM table was obtained from EpiRegio with the cell type/tissue filter applied (e.g. 'heart') before conversion to BED

## Limitations

- No tool versions are stated for any of the software mentioned (PASTAA, EpiRegio, bedtools)
- The linear model used in STITCHIT, which forms the basis for assessing the Cell type score used to rank the REMs, is a simplification of the actual regulation of genes by their REMs
- Sequence extraction is only meaningful if the genome FASTA assembly matches the coordinate system of the EpiRegio REM table

## Evidence

- [methods] we determine the DNA-sequence of the identified REMs using *bedtools*: "we determine the DNA-sequence of the identified REMs using *bedtools*"
- [methods] Download the resulting table by clicking on the bottom *CSV*.: "Download the resulting table by clicking on the bottom *CSV*."
- [readme] This repository contains examples and code to perform the Application scenarios explained in the EpiRegio documentation: [evidence span withheld — anchor status: not_found]
- [methods] If the Benjamini-Hochberg adjusted p-value from PASTAA is smaller than or equal to the parameter *<pvalue>* the motif is assumed to be significant enriched.: "If the Benjamini-Hochberg adjusted p-value from PASTAA is smaller than or equal to the parameter *<pvalue>* the motif is assumed to be significant enriched."
