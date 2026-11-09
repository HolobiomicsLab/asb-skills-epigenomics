---
name: chip-seq-quality-atlas-stratification
description: Use when compiling ChiLin QC outputs (e.g., id_frip FRiP JSON records)
  and you need to benchmark what proportion of ChIP versus input samples exceed a
  1% FRiP threshold.
when_to_use_negative:
- Input is raw FASTQ/BAM data rather than already-compiled FRiP QC records — run the
  ChiLin pipeline including step 7 (FRiP score calculation) first
- The goal is general QC of a single experiment (PBC, NSC/RSC, fragment size) rather
  than population-level FRiP threshold assessment across the quality atlas
- Samples cannot be reliably classified as ChIP treatment versus input control, since
  the stratified proportions depend on this distinction
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: ChiLin
  role: Pipeline whose step 7 calculates FRiP scores for each sample; its compile_database.sh
    script aggregates the QC records into the database used for threshold assessment
  repo: https://github.com/cfce/chilin
- name: MACS2
  role: Default peak caller in ChiLin; its peaks underlie the FRiP score calculation
    feeding the threshold assessment
  repo: https://github.com/taoliu/MACS/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s12859-016-1274-4
    title: 'ChiLin: a comprehensive ChIP-seq and DNase-seq quality control and analysis
      pipeline'
  source_content_hash: sha256:ebb34964d1d201fe0c777afd65a0b4912a74a5ab2933b72252ff16538bb4c774
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cfce/chilin
  tools:
  - ChiLin
  - MACS2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# chip-seq-quality-atlas-stratification

## Summary

Stratifies samples from the compiled ChiLin quality atlas (23,677 public ChIP-seq and DNase-seq samples) into ChIP (treatment) versus input (control) classes and evaluates them against a 1% FRiP score cutoff to validate FRiP as a ChIP enrichment reference metric.

## When to use

Use when compiling ChiLin QC outputs (e.g., id_frip FRiP JSON records) and you need to benchmark what proportion of ChIP versus input samples exceed a 1% FRiP threshold.

## When NOT to use

- Input is raw FASTQ/BAM data rather than already-compiled FRiP QC records — run the ChiLin pipeline including step 7 (FRiP score calculation) first
- The goal is general QC of a single experiment (PBC, NSC/RSC, fragment size) rather than population-level FRiP threshold assessment across the quality atlas
- Samples cannot be reliably classified as ChIP treatment versus input control, since the stratified proportions depend on this distinction

## Inputs

- Compiled ChiLin QC database of FRiP JSON records (e.g., id_frip) aggregated by compile_database.sh
- Sample metadata distinguishing ChIP (treatment) from input (control) samples
- Optionally the underlying BAM files and called peaks from ChiLin step 7 FRiP score calculation

## Outputs

- frip_cutoff_summary.csv with per-class pass counts and proportions (expected: 12,032/14,866 = 80.9% ChIP; 4,289/8,811 = 48.6% input)
- Per-sample FRiP scores classified as passing (>0.01) or failing the 1% threshold

## How to apply

Load the compiled QC database (FRiP JSON records such as id_frip, aggregated by the compile_database.sh post-processing script) covering the 23,677 public samples. Parse each sample's FRiP score from step 7 of the ChiLin pipeline and stratify records into ChIP (treatment) versus input (control) classes using sample metadata. Apply the 1% threshold, counting a sample as passing when its FRiP score is greater than 0.01, then compute per-class counts and proportions. Validate against the published reference values: 12,032/14,866 (80.9%) for ChIP and 4,289/8,811 (48.6%) for input; agreement within rounding supports 1% FRiP as a proper ChIP enrichment reference.

## Related tools

- **ChiLin** (Pipeline whose step 7 calculates FRiP scores for each sample; its compile_database.sh script aggregates the QC records into the database used for threshold assessment) — https://github.com/cfce/chilin
- **MACS2** (Default peak caller in ChiLin; its peaks underlie the FRiP score calculation feeding the threshold assessment) — https://github.com/taoliu/MACS/

## Examples

```
python -c "import ,csv; recs=.load(open('id_frip.')); r=[(c,float(s)) for c,s in recs]; ch=[x for x in r if x[0]=='ChIP']; w=open('frip_cutoff_summary.csv','w'); csv.writer(w).writerows([['class','pass','total','prop'],['ChIP',sum(f>0.01 for _,f in ch),len(ch),sum(f>0.01 for _,f in ch)/len(ch)]])"
```

## Evaluation signals

- ChIP class totals equal 14,866 samples and input class totals equal 8,811 samples
- Computed ChIP pass proportion matches 0.809 (12,032/14,866) within rounding
- Computed input pass proportion matches 0.486 (4,289/8,811) within rounding
- Every sample is counted exactly once in exactly one class (ChIP vs input), with no unclassified records
- frip_cutoff_summary.csv contains both raw counts and proportions per class

## Limitations

- Requires precompiled FRiP QC records; the threshold assessment cannot be run directly on raw sequencing data
- The 1% FRiP cutoff is validated for ChIP enrichment assessment; input controls show markedly lower pass rates (48.6%), so the threshold is not appropriate for judging control sample quality
- ChiLin is an extensible suite and the reproducibility discussion provides no concrete environment specification, so exact reproduction may depend on pinned tool versions (e.g., macs2=2.1.0 per the README)

## Evidence

- [results] 1% FRiP threshold supports ChIP enrichment assessment: [evidence span withheld — anchor status: not_found]
- [methods] FRiP calculation is ChiLin step 7: "step 7: calculate FRiP scores for each sample."
- [intro] compile_database.sh aggregates QC records: "post-processing script for compiling QC database is provided at https://github.com/cfce/chilin/blob/master/demo/compile_database.sh"
- [abstract] Scale of the quality atlas: "over 23,677 public ChIP-seq and DNase-seq samples (11,265 datasets) from eight literature-based classified categories"
- [intro] MACS2 is the default peak caller: "ChiLin uses MACS2 (https://github.com/taoliu/MACS/) as the default peak caller"
