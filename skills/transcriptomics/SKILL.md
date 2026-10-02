---
name: transcriptomics
description: Analyse gene expression with smarts.bio. Use when the user has RNA-seq data, wants a count matrix or differential expression, asks where a gene is expressed across tissues, or asks about RNA secondary structure or codon usage.
---

# Transcriptomics

Two distinct jobs arrive under "expression": processing the user's own RNA-seq
reads, and looking up what's already known about where a gene is expressed. The
second needs no data at all.

## The user's own RNA-seq

`rna-seq-analysis` runs HISAT2 alignment plus FeatureCounts and produces a
**count matrix ready for DESeq2 or edgeR**. 1–3 hours.

It needs four inputs, and the two that trip people up are the annotation and the
reference:

- `fastq_r1_key`, `fastq_r2_key` — paired reads
- `gtf_key` — the gene annotation. **The GTF must match the reference build.**
  A GRCh38 GTF against a GRCh37 genome produces counts that look fine and mean
  nothing
- `reference` — the genome

If the user doesn't have a GTF or reference, that's the blocker to solve first —
see `public-data`.

Run `quality-control` first on data they haven't checked. Adapter contamination
and failed runs are much cheaper to find in 20 minutes than after three hours.

**Where the pipeline stops.** It produces counts. Differential expression —
DESeq2, edgeR — is the next step and not part of this pipeline. Say that plainly
rather than implying the result is a list of differentially expressed genes.

What you *can* do with the count matrix, via `smarts_analyze_file`:

- descriptive statistics per column, including how many values are missing
- **PCA** with variance explained — enough to see whether samples separate the
  way the user expects, which is the first thing to check
- correlation between samples, and outlier detection
- **group comparisons** — t-test or one-way ANOVA across a grouping column, with
  Bonferroni correction, adjusted p-values and Cohen's d

That last one is not DESeq2 and must not be presented as it: it has no
negative-binomial model, no dispersion estimation and no library-size
normalisation, so on raw counts it is exploratory at best. Normalised values and
a clear grouping column make it a reasonable screen. Say which it is.

There is no clustering step — don't offer one. `smarts_view_as_image` with
`chart_type: "heatmap-2d"` gives the visual that people usually mean by it, and
`visualization-generator` makes volcano and MA plots once fold changes exist.

## Expression without data

"Where is this gene expressed?" is a lookup, not a pipeline. `gtex-expression`
gives tissue-level human expression; confirm its parameters with
`smarts_get_tool` first. **Never invent a tool id.**

This is a good answer for someone with no data yet: a real result in seconds.

## RNA sequence questions

For an RNA sequence rather than a dataset, `smarts_analyze_file` gives
composition statistics, codon usage and a nucleotide BLAST — see
`sequence-analysis`. Codon usage matters for expression construct design.

**No secondary-structure folding and no miRNA target prediction**, despite both
being natural things to ask of RNA. If the user wants a fold, say it isn't
available here.

## Sanity checks worth running

When you have counts, a few things are worth saying unprompted:

- **Library sizes** that differ by more than a few fold across samples will
  distort comparisons
- **Samples that don't cluster by condition** in PCA usually means a swap or a
  batch effect, and it is better found now
- **A very low fraction of assigned reads** points at a GTF/reference mismatch
  or the wrong strandedness

## Honesty

Report the counts the pipeline produced. Don't infer fold changes that were
never computed, and don't describe expression patterns from memory — tissue
expression comes from the database or it doesn't get stated. If the smarts.bio
tools are unavailable or unauthorized, stop and ask the user to connect.
