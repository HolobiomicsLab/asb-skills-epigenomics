---
name: differential-expression-log2foldchange-scoring
description: Use when raw sequencing counts from paired control/condition anchored
  genomic regions must be converted to a continuous biological activity score for
  a score-based motif enrichment analysis such as MEPP.
when_to_use_negative:
- Sequences are uncentered or not anchored to a defined coordinate (TSS/cleavage site),
  since MEPP profiles motif enrichment relative to a central anchor point
- The analysis goal is de novo motif discovery — MEPP currently functions only with
  a fixed motif library
- Regions lack a graded activity signal (e.g. only binary differential calls), since
  MEPP scores regions across a continuum of biological interest rather than presence/absence
  labels
tools:
- name: MEPP
  role: Consumes the DESeq2 log2 fold change scores as a scored FASTA and produces
    positional motif enrichment profiles relative to the sequence center
  repo: https://github.com/npdeloss/mepp
- name: pip
  role: Installs MEPP for use in the analysis project
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:2f990d50db7c24320086701bc484f9b120cd3fe1905b424749860021a33a505b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - MEPP
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# differential-expression-log2foldchange-scoring

## Summary

Computes DESeq2 log2 fold changes of LPS-stimulated vs resting macrophage anchored regions (csRNA-seq TSSs, ATAC-seq Tn5 cleavage sites, H3K27ac MNase-seq cleavage sites from GSE203135) and uses them as the biological activity score continuum that MEPP requires for positional motif enrichment profiling.

## When to use

Use when raw sequencing counts from paired control/condition anchored genomic regions must be converted to a continuous biological activity score for a score-based motif enrichment analysis such as MEPP.

## When NOT to use

- Sequences are uncentered or not anchored to a defined coordinate (TSS/cleavage site), since MEPP profiles motif enrichment relative to a central anchor point
- The analysis goal is de novo motif discovery — MEPP currently functions only with a fixed motif library
- Regions lack a graded activity signal (e.g. only binary differential calls), since MEPP scores regions across a continuum of biological interest rather than presence/absence labels

## Inputs

- Raw csRNA-seq, ATAC-seq, and MNase-seq datasets for LPS-stimulated and resting BMDMs (GEO accession GSE203135)
- Anchored ±200 bp genomic sequences centered on csRNA-seq TSSs or ATAC-seq/MNase-seq cleavage sites, with read counts per condition
- Motif matrix file in JASPAR format (e.g. data/homer.motifs.txt from the mepp repository)

## Outputs

- DESeq2 log2 fold change per anchored region (LPS-stimulated vs resting)
- Scored FASTA file with headers of the form '>sequence_name sequence_score'
- MEPP positional enrichment profiles (png/svg plots plus clustering table with adjusted p-values) per assay and motif

## How to apply

Download raw csRNA-seq, ATAC-seq, and MNase-seq datasets for LPS-stimulated and resting BMDMs from GEO accession GSE203135, then extract ±200 bp genomic sequences centered on the 5'-end coordinates (transcription start sites for csRNA-seq, cleavage sites for ATAC-seq and MNase-seq). Run DESeq2 on counts over these anchored regions to obtain a log2 fold change (LPS-stimulated vs resting) per region. Write the sequences as a scored FASTA in which each header carries the sequence name and its log2 fold change, in the format MEPP expects ('>sequence_name sequence_score'). Supply this scored FASTA to MEPP (e.g. with the NF-kB motif from data/homer.motifs.txt) so enrichment is profiled across the score continuum rather than at an arbitrary threshold — the score-based principle avoids arbitrary threshold selection while controlling for sequence bias. Validate that resulting MEPP profiles show the expected concordant NF-kB peaks: 58 bp upstream of the csRNA-seq TSS, ~45 bp downstream of the Tn5 cleavage site, and 81 bp upstream of the MNase-seq cleavage site.

## Related tools

- **MEPP** (Consumes the DESeq2 log2 fold change scores as a scored FASTA and produces positional motif enrichment profiles relative to the sequence center) — https://github.com/npdeloss/mepp
- **pip** (Installs MEPP for use in the analysis project) — https://github.com/npdeloss/mepp

## Examples

```
mepp --fa lps_diff_scores.fa --motifs data/homer.motifs.txt --out mepp_nfkb_out
```

## Evaluation signals

- Every FASTA header in the scored input follows the '>sequence_name sequence_score' form, with the numeric score equal to the DESeq2 log2 fold change
- MEPP NF-kB positional profile for csRNA-seq shows a clear positive peak 58 bp upstream of the TSS
- NF-kB peaks are concordantly positive across all three assays: ~45 bp downstream of the ATAC-seq Tn5 cleavage site and 81 bp upstream of the MNase-seq cleavage site
- Permutation-based confidence intervals on the positional profile exclude zero at the reported peak positions (default --perms 1000, --ci 95.0)

## Limitations

- MEPP currently functions only with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified for MEPP and the repository has no changelog
- Permutation testing can lead to significant GPU memory usage; --gjobs must be set carefully to avoid jobs crowding each other out of GPU memory

## Evidence

- [other] Compute DESeq2 log2 fold change of each anchored region (LPS-stimulated vs resting) as the biological activity score continuum for the sequences.: [evidence span withheld — anchor status: not_found]
- [discussion] MEPP usable by scoring genomic regions across a continuum: "MEPP is usable by scoring genomic regions across a continuum of scores reflecting two extremes of biological interest."
- [intro] Score-based enrichment avoids arbitrary threshold selection: "The score-based principle of this enrichment method further avoids issues with arbitrary threshold selection, while controlling for sequence bias."
- [readme] Scored FASTA header format: [evidence span withheld — anchor status: external_doc]
