# bisulfite-dmr-analysis-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **align_bisulfite_reads** — align bisulfite-treated reads to the reference genome  →  `bisulfite-read-alignment--c6ac0ff26c82f263`, `wgbs-read-alignment-bisulfite-mode`, `bowtie2-based-short-read-alignment`, `bisulfite-read-alignment--a2d18d44f25cd691`
2. **extract_cpg_counts** — extract methylated and unmethylated read counts at individual CpG sites  →  `cpg-methylation-count-extraction`, `strand-merged-methylation-calling`, `bismark-sam-alignment-parsing`, `methylc-seq-alignment-processing`
3. **cluster_cpg_regions** — aggregate CpGs into regional total and methylated counts for each sample  →  `cpg-site-region-clustering`, `pairwise-group-methylation-comparison`, `reference-implementation-output-validation`
4. **call_differentially_methylated_regions** — test clustered regions for differential methylation between sample groups  →  `beta-binomial-differential-methylation-testing`, `dss-wald-test-workflow`, `methylc-seq-region-dmr-identification`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
