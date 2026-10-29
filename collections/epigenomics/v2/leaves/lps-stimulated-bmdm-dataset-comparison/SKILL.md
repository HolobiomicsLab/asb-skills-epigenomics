---
name: lps-stimulated-bmdm-dataset-comparison
description: Use when comparing motif positional enrichment across multiple genomics
  assays (csRNA-seq, ATAC-seq, MNase-seq) of a differential stimulation response (e.g.
  LPS vs resting BMDMs), or when validating cross-assay concordance of TF motif positional
  profiles.
when_to_use_negative:
- The goal is de novo motif discovery — MEPP currently only functions with a fixed
  motif library and cannot assemble motifs from scratch
- Motif enrichment is needed independent of position within sequences (standard ZOOPs-style
  MEA such as HOMER or MEIRLOP suffices)
- Input sequences cannot be centered on an anchor point (e.g. TSS or cleavage site),
  since MEPP profiles are defined relative to such key anchor points
edam_operation: http://edamontology.org/operation_0445
edam_topics:
- http://edamontology.org/topic_3179
- http://edamontology.org/topic_0622
tools:
- name: MEPP
  role: Generates the positional enrichment profiles of the NF-kB motif across the
    scored, anchor-centered sequences for each of the three assays
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installs MEPP from the GitHub repository (pip install mepp) or user-local
    install with --user
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:53e71e8789b48b849a9ed1d87cb29d65f32607cf255912048cf6613467217aa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: http://edamontology.org/operation_0445
  edam_topics:
  - http://edamontology.org/topic_3179
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lps-stimulated-bmdm-dataset-comparison

## Summary

Reproduce concordant NF-kB positional enrichment profiles across csRNA-seq, ATAC-seq, and H3K27ac MNase-seq differential LPS analyses of BMDMs by running MEPP on score-anchored sequences from each assay (GEO GSE203135), testing whether motif positional peaks agree across measurements.

## When to use

Use when comparing motif positional enrichment across multiple genomics assays (csRNA-seq, ATAC-seq, MNase-seq) of a differential stimulation response (e.g. LPS vs resting BMDMs), or when validating cross-assay concordance of TF motif positional profiles.

## When NOT to use

- The goal is de novo motif discovery — MEPP currently only functions with a fixed motif library and cannot assemble motifs from scratch
- Motif enrichment is needed independent of position within sequences (standard ZOOPs-style MEA such as HOMER or MEIRLOP suffices)
- Input sequences cannot be centered on an anchor point (e.g. TSS or cleavage site), since MEPP profiles are defined relative to such key anchor points

## Inputs

- Raw csRNA-seq, ATAC-seq, and H3K27ac MNase-seq FASTQ/BAM data for LPS-stimulated and resting BMDMs (GEO GSE203135)
- Transcription start site coordinates (csRNA-seq anchors) and cleavage site coordinates (ATAC-seq Tn5, MNase-seq anchors)
- Scored FASTA file with headers of the form '>sequence_name sequence_score', where the score is the DESeq2 log2 fold change (LPS vs resting)
- Motif matrices file in JASPAR format containing the NF-kB motif

## Outputs

- MEPP positional enrichment profiles (png/svg plots) of the NF-kB motif relative to the assay-specific anchor point for each of the three assays
- Cross-assay comparison of NF-kB positional peak locations: +58 bp (csRNA-seq), -45 bp (ATAC-seq), +81 bp (MNase-seq) relative to anchors
- Clustering table with multiple-testing-adjusted positional correlation p-values (optional MEPP output)

## How to apply

Download raw csRNA-seq, ATAC-seq, and MNase-seq datasets for LPS-stimulated and resting BMDMs from GEO accession GSE203135. Extract ±200 bp genomic sequences centered on the 5'-end anchor coordinates: transcription start sites for csRNA-seq and cleavage sites for ATAC-seq and MNase-seq. Compute DESeq2 log2 fold change (LPS vs resting) for each anchored region and embed this score in the FASTA headers ('sequence_name sequence_score') so MEPP treats it as the biological activity continuum. Install MEPP via pip and run it with the NF-kB motif (JASPAR-format motif matrix) on each assay's scored FASTA to generate positional enrichment profiles. The analysis is validated if the profiles reproduce the concordant positive NF-kB peaks from Figure 6: 58 bp upstream of the csRNA-seq TSS, ~45 bp downstream of the Tn5 cleavage site, and 81 bp upstream of the MNase-seq cleavage site.

## Related tools

- **MEPP** (Generates the positional enrichment profiles of the NF-kB motif across the scored, anchor-centered sequences for each of the three assays) — https://github.com/npdeloss/mepp
- **pip** (Installs MEPP from the GitHub repository (pip install mepp) or user-local install with --user) — https://github.com/npdeloss/mepp

## Examples

```
pip install mepp; mepp --fa bmdm_lps_deseq2_scored.fa --motifs nfkb_jaspar.txt --out nfkb_csRNA_profile --center 200 --perms 1000
```

## Evaluation signals

- NF-kB MEPP profile for csRNA-seq shows a positive positional peak at 58 bp upstream of the TSS in the differential LPS analysis
- NF-kB MEPP profile for ATAC-seq shows a peak ~45 bp downstream of the Tn5 cleavage site
- NF-kB MEPP profile for MNase-seq shows a peak 81 bp upstream of the cleavage site
- All three assays yield concordantly positive NF-kB peaks in the differential LPS MEPP profiles, reproducing Figure 6
- Scored FASTA headers parse correctly as '>sequence_name sequence_score'; rejected sequences exceed the --dgt degenerate-base percentage threshold

## Limitations

- MEPP currently only functions with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version specified for MEPP and no changelog found for the repository
- Permutation testing (--perms, default 1000) can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory, causing profile generation to fail
- Findings are specific to the LPS differential response in BMDMs measured by these three assays; peak positions may not transfer to other cell types or stimuli

## Evidence

- [supplementary] All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135: "All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135"
- [methods] pip install command: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
- [readme] MEPP definition: "Motif Enrichment Positional Profiling (MEPP) quantifies a positional profile of motif enrichment along the length of DNA sequences centered on e.g. transcription start sites"
- [readme] Scored FASTA header format: "--fa TEXT                       Path to a scored fasta file, where sequence
                                      headers are of the form: ">sequence_name"
- [abstract] MEPP utility finding: "we demonstrate MEPP's utility in determining the sequence positions where motif presence correlates with measures of biological activity"
