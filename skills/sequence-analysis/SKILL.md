---
name: sequence-analysis
description: Analyse DNA, RNA or protein sequences with smarts.bio. Use when the user pastes or uploads a sequence, has a FASTA or FASTQ file, asks what a sequence is, wants GC content, ORFs, CpG islands, codon usage or secondary structure, or wants to search for homologs with BLAST.
---

# Sequence analysis

The fastest way to show smarts.bio doing something real: a pasted sequence needs
no upload link and returns in seconds.

## Get it into the workspace first

A sequence in the conversation goes straight in with `smarts_upload_file`. Give
it a `.fasta` extension so it's recognised as a sequence. A FASTQ on the user's
machine needs `smarts_request_upload` instead.

## Then analyse it

`smarts_analyze_file` detects DNA, RNA or protein automatically and runs what
fits:

- **DNA** — composition and GC content, open reading frames, CpG islands,
  restriction sites, nucleotide BLAST
- **RNA** — composition, codon usage, nucleotide BLAST
- **Protein** — composition, domain annotation (InterPro), UniProt records,
  pathways, interactions (STRING), protein BLAST

If detection is wrong — it happens with short or ambiguous sequences — pass
`{ "sequence_type": "protein" }` in `parameters` rather than accepting a bad
analysis.

**What it does not do:** no RNA secondary-structure folding and no miRNA target
prediction. Don't offer either. If the user needs a fold, say it isn't available
here rather than producing something adjacent.

## The homology search decision

**Homology search is off by default, and that is usually right.**

It queries external databases, takes minutes rather than seconds, and the AI
interpretation only arrives after it returns. So it dominates the wait for the
whole analysis.

**Turn it on when identity is the question:** "what is this sequence", "where
does this come from", "what organism", "find similar proteins". Then say it will
take a few minutes.

**Leave it off for composition questions:** GC content, ORFs, CpG islands,
restriction sites, codon usage, length and quality. These return in seconds.

**For a protein, there is a fast alternative worth knowing about.** The
`smartsmatch-protein` tool searches by vector embedding and returns similar
proteins in under a second, where BLAST takes minutes. Find it with
`smarts_list_tools`, inspect it with `smarts_get_tool`, run it with
`smarts_run_tool`. Use it when the user wants a quick identification or a
neighbourhood; use BLAST when they need alignments, E-values and coverage they
can defend in a paper.

If the analysis hits its time budget it returns what finished plus what was
still running — usually the homology search. Report that honestly rather than
presenting a partial analysis as complete.

## FASTQ is not FASTA

FASTQ carries per-base quality and is usually sequencing reads — often millions
of them. For a FASTQ the useful first move is the `quality-control` pipeline,
not sequence analysis. Analysing one read out of ten million answers nothing.

Large files are read from the start, so composition statistics describe the
prefix rather than the whole file. Say so when it matters.

## Show it as well as describe it

`smarts_view_file` opens a sequence viewer in the conversation — the user can
see features and composition rather than reading your summary of them.

## Beyond one sequence

`smarts_analyze_file` works on one sequence at a time. For anything comparative,
the tools are the route — discover with `smarts_list_tools`:

- **Multiple sequence alignment and phylogeny** — `ebi-job-dispatcher` reaches
  50+ EMBL-EBI tools including Clustal and InterProScan. There is no MSA
  pipeline; this is how alignment across sequences gets done
- **Editing and extraction** — `sequence-editor-toolkit` for reverse complement,
  translation, trimming, subsequences
- **Format problems** — `format-conversion-toolkit`; see `data-wrangling`

A resulting tree has no inline viewer, but `smarts_analyze_file` does analyse
Newick trees, and `smarts_open_file` shows one in the full app.

## Where to go next

Sequence analysis is often the first step, not the answer:

- Protein, and the user wants function → `protein-function`
- Protein, and the user wants a structure → `protein-structure`
- A gene, and the user wants the literature → `literature-and-evidence`
- A reference sequence they don't have yet → `public-data`

## Honesty

Report what the analysis computed. Never state GC content, ORF counts or
homologs from inspection or memory — they come from a tool result or they don't
get stated. If the smarts.bio tools are unavailable or unauthorized, stop and
ask the user to connect their account.
