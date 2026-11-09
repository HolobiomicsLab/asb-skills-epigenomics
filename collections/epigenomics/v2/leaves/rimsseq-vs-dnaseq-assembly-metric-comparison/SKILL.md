---
name: rimsseq-vs-dnaseq-assembly-metric-comparison
description: Use when paired-end RIMS-seq reads (e.g. from NCBI BioProject PRJNA706563)
  must be shown to support de novo genome assembly at quality metrics equivalent to
  standard Illumina DNA-seq shotgun reads.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or another non-adapter-ligation
  protocol (RIMS-seq does not work in that case)
- An exact reference genome is available and only methylation-motif identification
  is needed — the README's option 2 makes SPAdes assembly unnecessary
- Reads were downsampled below the 1–5 million paired-end range recommended by the
  pipeline, compromising assembly contiguity
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: SPAdes
  role: De novo assembles the adapter/quality-trimmed RIMS-seq and DNA-seq(+3H) paired-end
    read sets into contigs FASTA files for metric comparison
  repo: https://github.com/ablab/spades
- name: Quast
  role: Evaluates both SPAdes assemblies and extracts the mismatch rate and N50 used
    for the RIMS-seq vs control comparison
- name: RIMS-seq.pl
  role: Pipeline entry point that includes the SPAdes genome assembly step used to
    generate the assembled genome for downstream analysis
  repo: https://github.com/Ettwiller/RIMS-seq/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:c094f50c4bcec6517cba6c7174ec4b892ec26f5c55050307f2be31fed65fc95b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - Quast
  - RIMS-seq.pl
  - SPAdes
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# rimsseq-vs-dnaseq-assembly-metric-comparison

## Summary

Benchmarks whether RIMS-seq (alkaline-treated) paired-end reads yield a de novo bacterial genome assembly of quality comparable to a DNA-seq(+3H) control, by assembling both read sets with SPAdes and comparing Quast metrics such as mismatch rate and N50.

## When to use

Use when paired-end RIMS-seq reads (e.g. from NCBI BioProject PRJNA706563) must be shown to support de novo genome assembly at quality metrics equivalent to standard Illumina DNA-seq shotgun reads.

## When NOT to use

- The sequencing library was prepared by tagmentation or another non-adapter-ligation protocol (RIMS-seq does not work in that case)
- An exact reference genome is available and only methylation-motif identification is needed — the README's option 2 makes SPAdes assembly unnecessary
- Reads were downsampled below the 1–5 million paired-end range recommended by the pipeline, compromising assembly contiguity

## Inputs

- RIMS-seq (alkaline-treated) paired-end FASTQ reads for A. calcoaceticus ATCC 49823 (PRJNA706563)
- DNA-seq(+3H) control paired-end FASTQ reads for the same strain (PRJNA706563)

## Outputs

- Contigs FASTA for the RIMS-seq library (SPAdes assembly)
- Contigs FASTA for the DNA-seq(+3H) control library (SPAdes assembly)
- Quast report with mismatch rate and N50 per assembly
- quast_assembly_comparison.tsv with RIMS-seq vs control metrics side by side

## How to apply

Download the RIMS-seq (alkaline-treated) and DNA-seq(+3H) control paired-end FASTQ reads for Acinetobacter calcoaceticus ATCC 49823 from BioProject PRJNA706563, then apply adapter and quality trimming to both libraries (adapters must be removed before downstream RIMS-seq scripts). Assemble each trimmed read set de novo with SPAdes, which the RIMS-seq pipeline uses for its genome assembly step, producing a contigs FASTA per library. Evaluate both assemblies with Quast and extract the mismatch rate and N50 for each. Tabulate RIMS-seq vs control metrics side by side (quast_assembly_comparison.tsv); the validation criterion is that the RIMS-seq assembly's mismatch rate and N50 remain comparable to the DNA-seq(+3H) control, consistent with the paper's claim that alkaline treatment leaves these assembly metrics unchanged (Figure 3B, Supplementary Table S1).

## Related tools

- **SPAdes** (De novo assembles the adapter/quality-trimmed RIMS-seq and DNA-seq(+3H) paired-end read sets into contigs FASTA files for metric comparison) — https://github.com/ablab/spades
- **Quast** (Evaluates both SPAdes assemblies and extracts the mismatch rate and N50 used for the RIMS-seq vs control comparison)
- **RIMS-seq.pl** (Pipeline entry point that includes the SPAdes genome assembly step used to generate the assembled genome for downstream analysis) — https://github.com/Ettwiller/RIMS-seq/

## Examples

```
spades.py --pe1-1 rims_R1.trim.fq.gz --pe1-2 rims_R2.trim.fq.gz -o rims_spades && quast.py -o quast_out rims_spades/contigs.fasta control_spades/contigs.fasta
```

## Evaluation signals

- RIMS-seq assembly mismatch rate is comparable to the DNA-seq(+3H) control assembly (Supplementary Table S1)
- RIMS-seq assembly N50 is comparable to the DNA-seq(+3H) control assembly (Figure 3B)
- Both assemblies are non-empty SPAdes contigs FASTA files built from independently trimmed read sets
- quast_assembly_comparison.tsv contains side-by-side Quast metrics for exactly the two libraries

## Limitations

- The RIMS-seq library must be adapter-ligation based; tagmentation-based protocols are unsupported
- SPAdes (and BWA) are optional in the pipeline — if an exact reference genome exists and you perform your own mapping, the assembly step is unnecessary
- If the exact reference genome is not available, the README recommends option 1, where the genome is derived from the reads via assembly

## Evidence

- [methods] SPAdes is the pipeline assembler: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [abstract] RIMS-seq quality equals DNA-seq, enabling assembly substitution: "the resulting sequencing quality is identical to DNA-seq, enabling RIMS-seq to substitute standard sequencing of bacterial genomes"
- [readme] Reference-genome alternative to assembly: "if you have a reference genome and perform you own mapping, SPAdes and BWA are not necessary (see option 2)."
- [readme] Assembly is required when no exact reference exists: "If the exact reference genome is not available, we recommend to use option1, where the genome is derived from the"
