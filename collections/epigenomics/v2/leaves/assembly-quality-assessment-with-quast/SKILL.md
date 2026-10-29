---
name: assembly-quality-assessment-with-quast
description: Use when two shotgun read sets (e.g. a chemically treated library such
  as RIMS-seq and its untreated control) have each been assembled de novo and you
  must quantify whether treatment altered assembly quality via N50 and mismatch rate.
when_to_use_negative:
- The sequencing library was prepared by tagmentation or other non-adapter-ligation
  protocols — RIMS-seq data themselves are not valid inputs in that case
- An exactly matching reference genome is available and the analysis will be mapping-based
  rather than de novo assembly (the RIMS-seq pipeline's option 2 path, where SPAdes
  assembly is not needed)
- The question concerns methylase specificity discovery (motif calling) rather than
  assembly-quality metrics
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Quast
  role: Evaluates both SPAdes assemblies in one run, extracting the mismatch rate
    and N50 compared side by side in quast_assembly_comparison.tsv
- name: SPAdes
  role: De novo assembles each trimmed paired-end read set to produce the contigs
    FASTA that Quast evaluates
  repo: https://github.com/ablab/spades
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkab705
    title: Rapid identification of methylase specificity (RIMS-seq) jointly identifies
      methylated motifs and generates shotgun sequencing of bacterial genomes
  source_content_hash: sha256:a0088937f1a62842dc71402269d7e0badea08733f404b277400d8744f9382bb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ablab/spades
  tools:
  - Quast
  - SPAdes
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# assembly-quality-assessment-with-quast

## Summary

Evaluates de novo bacterial genome assemblies produced from paired-end reads (e.g. RIMS-seq alkaline-treated vs DNA-seq(+3H) control) by running Quast on each SPAdes contigs FASTA and comparing key metrics — specifically N50 and mismatch rate — to establish that a modified library protocol did not degrade assembly quality.

## When to use

Use when two shotgun read sets (e.g. a chemically treated library such as RIMS-seq and its untreated control) have each been assembled de novo and you must quantify whether treatment altered assembly quality via N50 and mismatch rate.

## When NOT to use

- The sequencing library was prepared by tagmentation or other non-adapter-ligation protocols — RIMS-seq data themselves are not valid inputs in that case
- An exactly matching reference genome is available and the analysis will be mapping-based rather than de novo assembly (the RIMS-seq pipeline's option 2 path, where SPAdes assembly is not needed)
- The question concerns methylase specificity discovery (motif calling) rather than assembly-quality metrics

## Inputs

- Paired-end FASTQ reads for the treated library (RIMS-seq, alkaline-treated) from BioProject PRJNA706563
- Paired-end FASTQ reads for the control library (DNA-seq(+3H))
- Adapter- and quality-trimmed paired-end reads for each library
- SPAdes contigs FASTA for the RIMS-seq library
- SPAdes contigs FASTA for the DNA-seq(+3H) control library

## Outputs

- Quast assembly report per contigs FASTA (containing N50 and mismatch rate)
- Side-by-side metric comparison table: quast_assembly_comparison.tsv

## How to apply

Download the paired-end FASTQ reads for both conditions (here, RIMS-seq and DNA-seq(+3H) reads of Acinetobacter calcoaceticus ATCC 49823 from BioProject PRJNA706563), perform adapter and quality trimming on each set, and assemble each trimmed set independently with SPAdes to produce one contigs FASTA per library. Run Quast on both contigs FASTAs (optionally against the reference genome) in a single invocation so metrics are computed identically for both assemblies, and extract the N50 and mismatch rate for each. Tabulate the RIMS-seq vs control metrics side by side (quast_assembly_comparison.tsv). Validation criterion from the paper: the RIMS-seq assembly's mismatch rate and N50 must be comparable to the DNA-seq(+3H) control assembly, consistent with Figure 3B and Supplementary Table S1 showing alkaline treatment leaves these assembly metrics unchanged.

## Related tools

- **Quast** (Evaluates both SPAdes assemblies in one run, extracting the mismatch rate and N50 compared side by side in quast_assembly_comparison.tsv)
- **SPAdes** (De novo assembles each trimmed paired-end read set to produce the contigs FASTA that Quast evaluates) — https://github.com/ablab/spades

## Examples

```
quast.py rims_contigs.fasta dna3h_contigs.fasta -o quast_assembly_comparison
```

## Evaluation signals

- Quast report contains both an N50 value and a mismatch rate for each of the two assemblies (RIMS-seq and DNA-seq(+3H) control)
- quast_assembly_comparison.tsv holds the RIMS-seq and control metrics side by side in a comparable layout
- RIMS-seq assembly mismatch rate is comparable to the DNA-seq(+3H) control's mismatch rate (no degradation from alkaline treatment)
- RIMS-seq assembly N50 is comparable to the control assembly's N50, consistent with Figure 3B and Supplementary Table S1
- Both assemblies were generated from equally trimmed read sets so metric differences reflect the library treatment, not preprocessing

## Limitations

- The paper's validation of unchanged assembly metrics is specific to mismatch rate and N50 (Figure 3B, Supplementary Table S1); other Quast metrics were not asserted comparable
- Assembly quality comparison requires a suitable control library sequenced and assembled under matched conditions; conclusions do not transfer to unrelated library preparations
- RIMS-seq data are only valid if the library used adapter ligation, so assemblies from tagmentation-based protocols cannot be assessed with this comparison

## Evidence

- [intro] RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing.: "RIMS-seq de novo identifies novel activities without the need for a reference genome and permits the assembly of the bacterial genome at metrics comparable to standard shotgun sequencing."
- [methods] SPAdes assembly step in the RIMS-seq pipeline: "It includes a genome assembly step using SPAdes and uses this assembled genome for the downstream analysis."
- [supplementary] BioProject accession for the source reads: "The data have been deposited with links to BioProject accession number PRJNA706563 in the NCBI BioProject database"
- [readme] SPAdes applicability to the input reads: [evidence span withheld — anchor status: external_doc]
