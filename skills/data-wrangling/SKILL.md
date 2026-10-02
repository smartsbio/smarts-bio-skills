---
name: data-wrangling
description: Convert, inspect and summarise data files in a smarts.bio workspace. Use when the user needs a file in a different format, has a CSV or table to explore, wants descriptive statistics, PCA or clustering, needs to extract an archive, or asks what is inside a file.
---

# Data wrangling

The unglamorous steps between having a file and getting an answer. Format
conversion, inspection, summary statistics. Most sessions touch this even when
it isn't what the user asked for.

## Find the tools first

Call `smarts_list_tools`, then `smarts_get_tool` for the one you pick.
**Never invent a tool id.** The ones that come up most:

| Need | Tool |
|---|---|
| Convert sequence, alignment, variant, annotation formats | `format-conversion-toolkit` |
| Extract or create archives | `zip-toolkit` |
| Read or parse a file server-side | `file-reader` |
| Find files in a workspace | `file-search` |
| Filter, join, reshape a table declaratively | `data-operations` |
| Statistics over a data file | `data-statistics` |
| Charts and plots saved to the workspace | `visualization-generator` |
| Write a report or document into the workspace | `file-writer` |
| Filter or summarise a VCF | `vcftools-toolkit` |
| Interval overlaps, nearest-feature, region queries | `bedtools-toolkit`, `genomicranges-toolkit`, `tabix-query` |

## Know what you have before converting

Check with `smarts_list_files`, then look:

- `smarts_view_file` renders tables, JSON, sequences and more interactively
- `smarts_read_file` returns text contents when you need to reason over them
- `smarts_analyze_file` gives the statistics (below)
- `smarts_view_as_image` with `chart_type` plots a CSV without writing any code —
  `bar-v`, `bar-stacked`, `line`, `area`, `scatter`, `bubble`, `pie`, `donut`,
  `heatmap-2d`, `boxplot`, `violin`, `hist`, `density`. Omit it to auto-pick. This
  is the fastest way to answer "show me this data", and you can see the result too

**Extension is a claim, not a fact.** A `.txt` that is really a FASTA, or a
`.csv` that is tab-separated, causes failures downstream that look like tool
bugs. Looking first is cheap.

## Conversion

Common needs: FASTQ↔FASTA, SAM↔BAM, VCF↔BCF, GFF↔GTF, Excel→CSV, archives out.

Two things to carry into the conversation:

- **Conversions can lose information.** FASTQ to FASTA discards quality scores,
  and there is no way back. Say so before doing it
- **Compressed is often fine.** Many tools read `.gz` directly, so decompressing
  a large file to feed a pipeline may be unnecessary work

## Tables

`smarts_analyze_file` on a CSV or TSV gives, in one call:

- **descriptive statistics** — rows, columns, which are numeric or categorical,
  missing values per column, and mean/std/min/max/median per numeric column
- **correlation** — the strongest positive and negative column pairs
- **outliers** — per column and total affected rows
- **PCA** — components and variance explained
- **group comparisons** — t-test or one-way ANOVA across a grouping column, with
  Bonferroni-adjusted p-values and Cohen's d
- an interpretation tying them together

For most "what's in this table" questions that is the whole answer.

**There is no clustering.** Don't offer it. A `heatmap-2d` chart or PCA is
usually what the user actually wanted.

Worth checking and reporting, because they change what the numbers mean:

- **Delimiter and header** — a header row read as data poisons every statistic
- **Missing values** — how many and where, before any summary is trusted
- **Mixed types in a column** — usually a data entry problem worth surfacing

Large files are read from the start, so statistics describe the beginning of the
file rather than all of it. Say so when the user is drawing conclusions from
them.

## Archives

Extract into the workspace, then list what arrived — users frequently don't know
what's inside, and the contents determine what to do next.

## Don't do by hand what a tool does

If a conversion or statistic has a tool, run it. Computing statistics by reading
a file's contents and doing arithmetic in your head is slower, unverifiable, and
wrong often enough to matter. The same goes for reformatting a file by writing
out a transformed copy — use the converter.

## Honesty

Report what the tools returned. **Never state statistics, row counts or file
contents you did not read**, and if a conversion failed, say so rather than
describing the output file. If the smarts.bio tools are unavailable or
unauthorized, stop and ask the user to connect their account.
