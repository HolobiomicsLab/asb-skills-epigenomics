# chipseq-peak-motif-discovery-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **align_and_assess_reads** — align ChIP-seq treatment and control reads and report early alignment quality  →  `fastq-read-alignment-quality-assessment`, `bwa-read-alignment-and-bam-processing`, `reference-genome-data-preparation`, `chipseq-quality-metric-computation`
2. **call_control_aware_peaks** — call enriched ChIP-seq peaks against the matched control  →  `chip-seq-peak-calling-with-control`, `chipseq-peak-calling-with-macs`, `chipseq-peak-calling--2dd556ad6f61330a`, `macs2-peak-calling`
3. **construct_motif_fastas** — construct peak-sequence and matched-background FASTA files  →  `discriminative-positive-negative-dataset-construction`, `chipseq-fasta-dataset-handling`
4. **discover_overrepresented_motifs** — identify motifs enriched in ChIP-seq peak sequences over matched background  →  `chipseq-overrepresented-motif-discovery`, `dna-motif-overrepresentation-analysis`, `fisher-exact-test-motif-significance`, `meme-format-motif-output`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
