---
name: atac-seq-differential-accessibility-scoring
description: Use when you have ATAC-seq (or similar accessibility) data for two cell
  types and want sequences ±100 bp around an anchor motif scored by a log2 fold-change
  so downstream motif analysis can test whether motif spacing correlates with cell-type-specific
  accessibility.
when_to_use_negative:
- You only need motif presence/absence enrichment without positional information —
  bin- or quartile-based MEA tools (CentriMo, TFEA) or a ZOOPs-model tool like HOMER
  suffice
- 'You need de novo motif discovery: MEPP currently only functions with a fixed motif
  library and de novo motif recognition/assembly is left to future work'
- Your sequences are not anchored on a defined feature (TSS or known motif) — the
  positional profile requires a biologically meaningful anchor point for the correlation
  to be interpretable
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3168
tools:
- name: MEPP
  role: Consumes the accessibility-scored, GATA1-anchored FASTA and outputs a positional
    enrichment profile that recovers SCL-to-GATA1 motif spacing associated with K562-specific
    accessibility
  repo: https://github.com/npdeloss/mepp
- name: CentriMo
  role: Baseline bin-based central enrichment comparison run on the same scored bins
    to validate ranking of SCL/bHLH motifs
- name: TFEA
  role: Baseline quartile-based comparison tool; fails to rank SCL/bHLH motifs at
    the top, illustrating the advantage of positional profiling
- name: HOMER
  role: Baseline comparison on top-10% bins under a zero-or-one-occurrence-per-sequence
    (ZOOPs) model
- name: pip
  role: Installation route for MEPP ('pip install mepp' or 'pip install git+https://github.com/npdeloss/mepp@main')
  repo: https://github.com/npdeloss/mepp
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nargab/lqac075
    title: 'MEPP: more transparent motif enrichment by profiling positional correlations'
  source_content_hash: sha256:77d0acafbee3275951cfc5ae5e45bf8a1df5cd71dfb03b135b58805191ddd073
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/npdeloss/mepp
  tools:
  - CentriMo
  - HOMER
  - MEPP
  - TFEA
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# atac-seq-differential-accessibility-scoring

## Summary

Compute a continuous differential chromatin accessibility score (e.g. log2 fold change of HCT116-vs-K562 ATAC-seq signal) for short genomic windows anchored on transcription factor motifs (GATA1), so that position-aware motif enrichment methods like MEPP can correlate motif position with cell-type-specific accessibility.

## When to use

Use when you have ATAC-seq (or similar accessibility) data for two cell types and want sequences ±100 bp around an anchor motif scored by a log2 fold-change so downstream motif analysis can test whether motif spacing correlates with cell-type-specific accessibility.

## When NOT to use

- You only need motif presence/absence enrichment without positional information — bin- or quartile-based MEA tools (CentriMo, TFEA) or a ZOOPs-model tool like HOMER suffice
- You need de novo motif discovery: MEPP currently only functions with a fixed motif library and de novo motif recognition/assembly is left to future work
- Your sequences are not anchored on a defined feature (TSS or known motif) — the positional profile requires a biologically meaningful anchor point for the correlation to be interpretable

## Inputs

- ATAC-seq raw data for two cell types (GEO accession GSE203135; ENCODE human cell line data)
- GATA1-anchored ±100 bp sequence windows (FASTA)
- A JASPAR-format motif matrices file (or data/homer.motifs.txt from the MEPP repo) containing the SCL/TAL1 and GATA1 motifs

## Outputs

- A scored FASTA where each header is '>sequence_name sequence_score' (log2 fold-change accessibility)
- MEPP positional enrichment profile plots (PNG/SVG) per motif relative to the GATA1 anchor
- MEPP clustering table with positionally adjusted p-values
- Compiled motif ranking table comparing MEPP, CentriMo, TFEA, and HOMER (reproducing Table S5)

## How to apply

Download ATAC-seq data for the two cell lines (GSE203135 or ENCODE human cell line data) and, for each ±100 bp window centered on a GATA1 motif occurrence, compute a log2 fold-change accessibility score between HCT116 and K562. Write the windows as a scored FASTA whose headers carry the score in the form '>sequence_name sequence_score', the exact input format MEPP expects. Run MEPP on this scored FASTA with a JASPAR motif matrix file; MEPP profiles positional motif enrichment relative to the GATA1 anchor while accounting for lower-order nucleotide bias. For validation, run the baseline MEA tools on the same scored bins (CentriMo bin-based, TFEA quartile-based, HOMER top-10% bins under a ZOOPs model) and compile rankings into a comparison table (Table S5). The decisive validation signal is the SCL (TAL1) motif's positional peak ~12 bp upstream of the GATA1 anchor, enriched toward K562-specific accessibility, with MEPP and CentriMo — but not TFEA or HOMER — ranking SCL/bHLH motifs at the top.

## Related tools

- **MEPP** (Consumes the accessibility-scored, GATA1-anchored FASTA and outputs a positional enrichment profile that recovers SCL-to-GATA1 motif spacing associated with K562-specific accessibility) — https://github.com/npdeloss/mepp
- **CentriMo** (Baseline bin-based central enrichment comparison run on the same scored bins to validate ranking of SCL/bHLH motifs)
- **TFEA** (Baseline quartile-based comparison tool; fails to rank SCL/bHLH motifs at the top, illustrating the advantage of positional profiling)
- **HOMER** (Baseline comparison on top-10% bins under a zero-or-one-occurrence-per-sequence (ZOOPs) model)
- **pip** (Installation route for MEPP ('pip install mepp' or 'pip install git+https://github.com/npdeloss/mepp@main')) — https://github.com/npdeloss/mepp

## Examples

```
pip install git+https://github.com/npdeloss/mepp@main; mepp --fa gata1_scored.fa --motifs jaspar_motifs.txt --out mepp_out
```

## Evaluation signals

- The MEPP positional profile for the SCL (TAL1) motif shows an enrichment peak ~12 bp upstream of the GATA1 anchor with a negative-scored valley specific to K562
- In the comparison table, MEPP and CentriMo rank SCL/bHLH motifs at the top while TFEA and HOMER do not (reproducing Table S5)
- The scored FASTA is well-formed: every header carries '>sequence_name sequence_score' as MEPP requires
- MEPP's profile controls for lower-order nucleotide bias, and results recapitulate a known biology: ternary complex formation on a composite GATA:E-box motif
- Multiple testing on the clustering table passes the default thorough adjustment (fdr_by method, alpha=0.01) for positional correlation p-values

## Limitations

- MEPP currently functions only with a fixed motif library; de novo motif recognition/assembly is left to future work
- No software version is specified for MEPP and no changelog exists for the repository
- Permutation testing can lead to significant GPU memory usage; if no GPU is available, --nogpu must be set and processing falls back to CPU multiprocessing
- HOMER's comparison uses a zero-or-one-occurrence-per-sequence (ZOOPs) model, which cannot capture positional effects and underperforms on this spacing question

## Evidence

- [abstract] outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting: "outputs a positional enrichment profile of a given TF's binding motif relative to key anchor points (e.g. transcription start sites, or other motifs) within the analyzed sequences while accounting"
- [supplementary] All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135: "All raw data generated for this study can be accessed at NCBI Gene Expression Omnibus (GEO; https://www.ncbi.nlm.nih.gov/geo/) accession number GSE203135"
- [readme] Path to a scored fasta file, where sequence headers are of the form: ">sequence_name sequence_score".: [evidence span withheld — anchor status: external_doc]
- [discussion] we find that our method recovers the positional relevance of SCL-to-GATA1 motif spacing to K562 cells, a result supported by the previous characterization of ternary complex formation on a composite: "we find that our method recovers the positional relevance of SCL-to-GATA1 motif spacing to K562 cells, a result supported by the previous characterization of ternary complex formation on a composite"
- [methods] To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp: "To install Motif Enrichment Positional Profiling, run this command in your terminal:

    $ pip install mepp"
