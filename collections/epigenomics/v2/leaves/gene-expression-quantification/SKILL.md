---
name: gene-expression-quantification
description: Use when raw RNA-seq reads have been mapped to a reference genome (BAM
  produced via ContextMap 2 and samtools) and per-gene read counts are needed as input
  for differential expression analysis.
when_to_use_negative:
- Input is already a gene count table — skip quantification and go directly to differential
  expression testing with the DETest module
- The analysis goal is transcript- or isoform-level quantification rather than gene-feature
  read counting
- 'Reads are not yet mapped: quantification requires indexed BAM alignments, so run
  QC, ContextMap 2 mapping, and samtools conversion first'
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Watchdog
  role: Workflow management system orchestrating the QC, mapping, featureCounts quantification,
    RSeQC QC, and DETest differential expression steps
  repo: https://github.com/klugem/watchdog
- name: Java
  role: Runtime platform for Watchdog (JDK 11 or higher required per README; article
    states Java 1.8 or higher)
  repo: https://github.com/klugem/watchdog
- name: featureCounts
  role: Module used to quantify gene expression by counting reads for gene features
    from mapped BAM files
  repo: https://github.com/watchdog-wms/watchdog-wms-modules
- name: ContextMap 2
  role: Module used to map RNA-seq reads against the reference genome prior to counting
  repo: https://github.com/watchdog-wms/watchdog-wms-modules
- name: samtools
  role: Converts SAM output to BAM and indexes alignments for downstream featureCounts
    execution
  repo: https://github.com/watchdog-wms/watchdog-wms-modules
- name: RSeQC
  role: Module run for mapping quality control after quantification of the alignment
  repo: https://github.com/watchdog-wms/watchdog-wms-modules
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-018-2107-4
    title: Watchdog – a workflow management system for the distributed analysis of
      large-scale experimental data
  source_content_hash: sha256:376373233cd4b1f3623472194f53e3aa4afd80bdb6c34b5306d8131efdc177db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/klugem/watchdog
  tools:
  - ContextMap 2
  - Java
  - RSeQC
  - Watchdog
  - featureCounts
  - samtools
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-expression-quantification

## Summary

Quantify gene-level expression from mapped RNA-seq reads by counting reads overlapping gene features, as implemented in the Watchdog RNA-seq workflow's featureCounts module, producing a gene count table that feeds downstream differential expression testing.

## When to use

Use when raw RNA-seq reads have been mapped to a reference genome (BAM produced via ContextMap 2 and samtools) and per-gene read counts are needed as input for differential expression analysis.

## When NOT to use

- Input is already a gene count table — skip quantification and go directly to differential expression testing with the DETest module
- The analysis goal is transcript- or isoform-level quantification rather than gene-feature read counting
- Reads are not yet mapped: quantification requires indexed BAM alignments, so run QC, ContextMap 2 mapping, and samtools conversion first

## Inputs

- Mapped RNA-seq alignments in BAM format (from ContextMap 2 mapping followed by SAM-to-BAM conversion and indexing with samtools)
- Reference genome annotation with gene features
- FASTQ input reads (initial raw input to the QC and mapping steps)

## Outputs

- Gene count table (counts of reads per gene feature across samples)
- Mapping QC reports from the RSeQC module
- Differential expression result tables per method (limma, edgeR, DESeq, DESeq2) produced by the DETest module
- Execution report via reportGenerator.jar (optional)

## How to apply

Within the Watchdog workflow2_differential_gene_expression.xml, first perform read quality checks with the Watchdog QC module (FastQC-style), then map reads against the reference genome using the ContextMap 2 module, convert and index the alignments with samtools. Run the featureCounts module on the indexed BAM files to count reads for gene features, yielding a gene count table. Run mapping QC with the RSeQC module to validate alignment quality before relying on the counts. Finally, pass the count matrix to the DETest module, which applies limma, edgeR, DESeq, and DESeq2 to detect differentially expressed genes. Collect the count table and per-method DE result tables from the workflow output folder, optionally summarizing execution with reportGenerator.jar.

## Related tools

- **Watchdog** (Workflow management system orchestrating the QC, mapping, featureCounts quantification, RSeQC QC, and DETest differential expression steps) — https://github.com/klugem/watchdog
- **Java** (Runtime platform for Watchdog (JDK 11 or higher required per README; article states Java 1.8 or higher)) — https://github.com/klugem/watchdog
- **featureCounts** (Module used to quantify gene expression by counting reads for gene features from mapped BAM files) — https://github.com/watchdog-wms/watchdog-wms-modules
- **ContextMap 2** (Module used to map RNA-seq reads against the reference genome prior to counting) — https://github.com/watchdog-wms/watchdog-wms-modules
- **samtools** (Converts SAM output to BAM and indexes alignments for downstream featureCounts execution) — https://github.com/watchdog-wms/watchdog-wms-modules
- **RSeQC** (Module run for mapping quality control after quantification of the alignment) — https://github.com/watchdog-wms/watchdog-wms-modules

## Examples

```
./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml
```

## Evaluation signals

- The gene count table is produced as a file in the workflow output folder
- All workflow steps, including featureCounts quantification, complete without triggering Watchdog's error-detection system
- Per-method differential expression result tables (limma, edgeR, DESeq, DESeq2) are generated by the DETest module from the count matrix
- RSeQC mapping QC reports are generated and show no mapping problems before downstream testing
- The workflow is launched successfully with ./watchdog.sh -x path/to/workflow2_differential_gene_expression.xml after running ./helper_scripts/configureExamples.sh

## Limitations

- The differential gene expression example workflow requires bwa and ContextMap 2 to be installed and configured in advance
- Watchdog requires JDK 11 or higher per the README (article states Java 1.8 or higher), and the workflow designer depends on JavaFX SDK 11 or higher
- No specific version number is stated for the Watchdog software release in the article
- No specific gene-count thresholds or DE significance cutoffs are given in the provided context

## Evidence

- [other] Quantify gene expression with the featureCounts module and run mapping QC with the RSeQC module.: [evidence span withheld — anchor status: not_found]
- [intro] Counting of reads for gene features: "e.g read quality control, mapping against the genome, counting of reads for gene features"
- [abstract] Comprehensive RNA-seq workflow shipped with software: "a comprehensive workflow and modules for the analysis of RNA-seq experiments were implemented and are provided with the software"
- [other] DETest module evaluates limma, edgeR, DESeq, DESeq2 on count matrix: [evidence span withheld — anchor status: not_found]
- [readme] Example workflow requires bwa and ContextMap2: "workflow2_differential_gene_expression.xml - workflow performing a differential gene expression analysis on an example data set (needs bwa and ContextMap2 to be installed and configured)"
