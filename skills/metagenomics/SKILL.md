---
name: metagenomics
description: Work with microbial community sequencing on smarts.bio, and set expectations about what the platform can and cannot do with it. Use when the user has metagenomic or microbiome data, asks what organisms are in a sample, mentions 16S, shotgun metagenomics, taxonomic profiling or community composition.
---

# Metagenomics

**Say this early: smarts.bio has no taxonomic profiler.** There is no Kraken,
MetaPhlAn, QIIME, mOTUs or Bracken equivalent in the tool catalogue. You cannot
turn a bag of metagenomic reads into a community profile here.

That is the single most useful thing you can tell a user with a microbiome
dataset, and it has to come first. A confident-sounding plan that collapses at the
profiling step wastes their afternoon; an honest answer in the first reply lets
them bring the part the platform *is* good at.

Check the catalogue with `smarts_list_tools` before claiming otherwise — if a
profiler has been added since this was written, use it. But do not go looking for
one by guessing tool ids.

## What does work, and it is not nothing

Three real routes, in the order they usually matter:

**1. Their abundance table, after profiling elsewhere.** This is the strongest
option. If they already have a table from QIIME, Kraken, MetaPhlAn or a
collaborator, `smarts_analyze_file` on the CSV/TSV gives descriptive statistics
with per-column missingness, correlation between taxa, outlier detection, PCA,
and group comparisons by t-test or ANOVA with Bonferroni correction and effect
sizes. For "do my cases differ from my controls" that is the actual analysis,
and it is the question most people are really asking.

`smarts_view_as_image` with a `chart_type` plots it — stacked bars for
composition, heatmap for taxa across samples, boxplot by group.

**2. Identifying individual sequences.** For one 16S amplicon, a recovered
contig, or a handful of sequences, `ncbi-blast` against nucleotide databases
identifies them, and `ncbi-taxonomy` resolves names and lineage. This is genuine
and fast. It does not scale to a whole sample, and saying "I can identify these
twelve sequences but not profile the community" is an honest, useful offer.

**3. Quality control on the raw reads.** The `quality-control` pipeline (FastQC +
MultiQC, 10–30 min) works on metagenomic FASTQ like any other. Worth running
before they take the data elsewhere, because the problems below are cheaper to
find now.

For recovered genes, `protein-function` covers functional annotation.

## Establish the experiment type anyway

It changes what any downstream answer can mean, and users often don't state it:

- **16S / amplicon** — one marker gene amplified. Cheap, genus-level
  composition, cannot tell you what the community is *doing*
- **Shotgun metagenomics** — everything sequenced. More expensive, resolves to
  species or strain, and carries functional information

A functional question against 16S data cannot be answered by anyone, and saying
so early saves the user more time than anything else in this skill.

## Quality control is not optional here

Metagenomic samples carry host contamination, adapter read-through and highly
variable depth, and all three distort composition rather than just adding noise.
A sample that is 80% host DNA will produce a confident, wrong community profile —
wherever that profile is generated.

## Reading a profile

When a profile exists — theirs, from elsewhere — these change the
interpretation and are worth stating unprompted:

- **Relative abundance is relative.** A taxon rising from 10% to 20% may have
  doubled, or everything else may have halved. Sequencing gives proportions, not
  absolute counts, and comparisons across samples inherit that
- **Read depth drives apparent diversity.** A shallow sample looks less diverse
  than a deep one from the same source. Compare at matched depth or say that you
  haven't
- **Unassigned reads are a result.** A large unclassified fraction usually means
  the reference database doesn't cover this environment, which is itself the
  finding — especially for soil and marine samples
- **Species-level calls from 16S are weak.** Genus is usually as far as the data
  supports
- **Compositional data breaks ordinary statistics.** The group comparisons in
  `smarts_analyze_file` assume independent columns, which proportions are not.
  Report them as exploratory and say that a compositional method (CLR transform,
  ALDEx2, ANCOM) is the rigorous version

## Where to go next

- Taxonomy and reference genomes → `public-data`
- Functional annotation of recovered genes → `protein-function`
- What's known about an organism found → `literature-and-evidence`
- Statistics on the abundance table → `data-wrangling`

## Honesty

Report the composition the data shows. **Do not name organisms you expect to be
present** — a plausible gut or soil community recalled from training reads
exactly like a real profile, and the user cannot tell them apart. And do not
describe a profiling run that the platform cannot perform: if the capability
isn't there, the answer is that it isn't there. If the smarts.bio tools are
unavailable or unauthorized, stop and ask the user to connect their account.
