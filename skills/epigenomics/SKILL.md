---
name: epigenomics
description: Analyse chromatin accessibility and protein-DNA binding with smarts.bio. Use when the user has ATAC-seq or ChIP-seq data, wants peak calling, asks about open chromatin, transcription factor binding sites, histone marks, or motif enrichment.
---

# Epigenomics

Two pipelines, for two different experiments. Both end in MACS2 peak calling,
and both are 1–3 hours.

## Which experiment

**`atac-seq`** — chromatin accessibility. Where is the genome open.

- Inputs: `fastq_r1_key`, `fastq_r2_key`, `reference`, `genome`
- Trimming → Bowtie2 → MACS2 peak calling → fragment size analysis
- 1–2 hours

**`chip-seq`** — where a specific protein binds. A transcription factor, a
histone mark.

- Inputs: `fastq_chip_key`, `fastq_input_key`, `reference`, `genome`
- Alignment → MACS2 peak calling → de novo motif enrichment with HOMER
- 1–3 hours

**ChIP-seq needs a control.** `fastq_input_key` is the input/IgG sample, and it
is not optional — peak calling without one produces peaks driven by chromatin
accessibility and sequencing bias rather than by binding. If the user has no
input sample, say that before running, not after.

`genome` and `reference` are separate inputs: the genome build identifier that
peak calling uses for effective genome size, and the sequence to align against.
Confirm both with `smarts_get_pipeline`.

## What the output tells you

Peaks come back as intervals. `smarts_view_file` opens them; `smarts_analyze_file`
gives descriptive statistics. Worth saying unprompted:

- **Fragment size distribution** is ATAC-seq's main quality signal. A clear
  nucleosome-free peak with periodic nucleosomal peaks behind it means the
  experiment worked. A smooth distribution without that structure usually means
  over-digestion or poor quality.
- **Peak counts** vary hugely by target. A sharp transcription factor gives
  thousands to tens of thousands; a broad histone mark gives far fewer, wider
  regions. A sharp factor returning a few hundred peaks usually means a failed
  ChIP, not a selective factor.
- **Motif enrichment** is the strongest confirmation in ChIP-seq. If the
  expected motif for the factor is the top hit, the experiment worked. If it
  isn't, say so — it is the most informative negative result the pipeline
  produces.

## Where the pipeline stops — and how to keep going

These pipelines produce peaks and motifs. Differential accessibility between
conditions, peak annotation to nearest genes, and comparison across experiments
are not part of them. Say so rather than implying the result answers a
differential question.

But the *platform* goes further than the pipeline does, and this is where most
users give up unnecessarily. Via `smarts_list_tools`:

- **`bedtools-toolkit`** — intersect, closest, merge, subtract. Peak-to-nearest-gene
  annotation, overlap between two conditions, peaks within promoters: all BEDTools
  operations against an annotation file
- **`genomicranges-toolkit`** — the Bioconductor equivalent, for interval
  arithmetic expressed as ranges
- **`tabix-query`** — pull a region out of a large indexed file instead of reading
  the whole thing
- **`homer-toolkit`** — motif enrichment directly, including on an ATAC-seq peak
  set, which the ATAC pipeline does not do for you
- **`ucsc-genome-browser`** — genomic context and tracks around a peak

So "which genes are near my peaks" and "which peaks are shared between my two
conditions" are both answerable today. They need an annotation file in the
workspace — see `public-data`, and mind the build.

A caveat worth checking rather than assuming: inline viewers cover `.bed`, but
MACS2 also emits `.narrowPeak`, `.broadPeak` and `_peaks.xls`, which have no
inline viewer and no analysis backend. List the outputs with `smarts_list_files`
and convert or rename to `.bed` if the user wants to look at them.

For peaks sitting near genes the user cares about, `literature-and-evidence` and
`protein-function` are the natural next steps.

## Honesty

Report the peaks and motifs the pipeline found. Don't estimate peak counts,
describe enrichment that wasn't computed, or name a motif from memory. If the
smarts.bio tools are unavailable or unauthorized, stop and ask the user to
connect their account.
