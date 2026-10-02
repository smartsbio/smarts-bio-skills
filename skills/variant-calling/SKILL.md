---
name: variant-calling
description: Find genetic variants from sequencing data with smarts.bio. Use when the user has FASTQ or BAM files and wants SNPs, indels, germline or somatic variants, tumour-normal calling, or asks to align reads before calling.
---

# Variant calling on smarts.bio

Turning reads into variants is a chain, not a single step. Most requests to
"find variants" start from FASTQ and need alignment first.

```
FASTQ → quality-control → alignment-wes / whole-genome-sequencing → BAM
BAM   → gatk-variant-calling (germline) or somatic-variant-calling (T/N) → VCF
```

The connector's own instructions cover the mechanics — discover, inspect, run,
poll. This skill is about choosing correctly.

## Three questions before you start

**1. What do they actually have?** FASTQ means the whole chain. A BAM means
calling only. Check with `smarts_list_files` rather than assuming from how they
describe it — "my sequencing data" means either.

**2. Germline or somatic?** This is the choice that matters most and users often
don't state it.

- **Germline** (`gatk-variant-calling`) — inherited variants, one sample.
  GATK4 HaplotypeCaller in GVCF mode.
- **Somatic** (`somatic-variant-calling`) — tumour-acquired variants, needs a
  tumour **and** a matched normal from the same individual. Mutect2 with
  artifact filtering.

If the user mentions cancer, tumour, or a matched normal, it is somatic. If they
have only one sample from a tumour, somatic calling is not available — say so
rather than silently running germline, which will report the patient's inherited
variants as if they were tumour mutations.

**3. Exome or genome?** `alignment-wes` for exome capture, 2–4 hours.
`whole-genome-sequencing` for WGS, 4–8 hours. Running WES alignment on WGS data
wastes most of the reads.

## Run QC first when the data is new

`quality-control` takes 10–30 minutes and catches adapter contamination, poor
quality tails and failed runs before a four-hour alignment. For data the user
has never looked at, it is worth the wait. For data they have already QC'd,
skip it.

**Check the BAM before calling from it**, too — this is cheap and skipped far too
often. `smarts_analyze_file` on a BAM returns the mapping rate, the flag
breakdown (paired, properly paired, duplicate, secondary, supplementary), the
MAPQ distribution and the insert size distribution. A low mapping rate or a
nonsensical insert size means the alignment is wrong, and finding that out before
a four-hour variant call rather than after is the whole point. There is no inline
viewer for BAM, but the analysis works and `smarts_view_as_image` renders it.

Single BAM operations — sort, index, mark duplicates, subset a region — are
`samtools-toolkit` and `picard-toolkit`, in minutes. Don't run a pipeline for them.

## Reading the VCF

Open the result — `smarts_view_file` renders a VCF in a variant viewer, and
`smarts_analyze_file` adds variant counts by class and chromosome, the Ts/Tv
ratio, the FILTER pass rate and the QUAL distribution.

It does **not** annotate consequences. "What does this variant do to the protein"
is a database question, not part of the analysis — see `variant-interpretation`.

**Ts/Tv is the quickest quality signal.** Roughly 2.0–2.1 for whole genome,
~3.0 for exome. Markedly lower suggests false positives. Say so when it's off;
it's the kind of thing a user wants told, not buried.

Variant counts worth sanity-checking: a human exome yields on the order of
20,000–50,000 variants, a whole genome 4–5 million. An order of magnitude out
means something upstream went wrong.

The analysis also reports the **FILTER pass rate** and the QUAL distribution. A
low pass rate is worth surfacing before the user starts interpreting calls.

To narrow a VCF — by region, quality, allele frequency, or down to a gene list —
use `vcftools-toolkit` rather than reading the file and reasoning over it.
Interpreting thousands of variants by hand is the wrong move; filter first, then
interpret the handful that matter.

## What this skill does not do

Calling a variant is not interpreting it. Once there's a VCF, "is this variant
pathogenic" is the `variant-interpretation` skill — ClinVar, ClinGen, dbSNP and
GWAS catalogue lookups — and it needs no pipeline.

## Honesty

Report what the pipeline produced. Don't estimate variant counts or quality
metrics that weren't computed, and if a run failed, say what failed rather than
describing the VCF it would have made. If the smarts.bio tools are unavailable
or unauthorized, stop and ask the user to connect.
