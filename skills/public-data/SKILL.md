---
name: public-data
description: Fetch reference genomes, annotations, public sequencing datasets and gene records through smarts.bio. Use when the user needs a reference genome or GTF, wants to find a published dataset, asks for a gene or transcript record, or needs taxonomy information.
---

# Public data and references

Getting the reference files a pipeline needs, and pulling records out of public
archives. This is usually a blocker rather than a goal: a user wanting RNA-seq
discovers they need a GTF.

## Find the tools first

Call `smarts_list_tools`, then `smarts_get_tool` for the one you pick.
**Never invent a tool id.**

| What's needed | Source |
|---|---|
| Reference genome, assembly metadata | `ncbi-assembly` |
| Gene record, transcripts, coordinates | `ensembl`, `ncbi-gene` |
| A sequence by accession | `ncbi-nucleotide`, `ncbi-protein` |
| Published raw sequencing data | `ncbi-sra` |
| Organism names and lineage | `ncbi-taxonomy` |
| Genomic context, tracks | `ucsc-genome-browser` |
| A region out of a large indexed file | `tabix-query` |
| A structure by accession | `pdb-toolkit`, `alphafold-db`, `ncbi-structure` |

## The build problem

Genome build is the single most common source of silent error, and it is worth
being pedantic about:

- **GRCh37/hg19 and GRCh38/hg38 coordinates differ by millions of bases.** The
  same position number means different places
- **The GTF must match the genome it annotates.** A mismatched pair produces
  counts that look fine and mean nothing — see `transcriptomics`
- **Chromosome naming differs** between sources: `chr1` versus `1`. Tools fail
  or silently find nothing when these disagree

When a user asks for "the human genome", ask which build if anything downstream
depends on it. When they give coordinates without a build, ask.

## Getting it into the workspace

Fetching a record returns data; it isn't in the workspace until it's saved
there. Write it with `smarts_upload_file` before using it as a pipeline input.

Reference genomes are large — often far beyond the inline write limit. If a
pipeline accepts a named reference rather than a file key, use that; check with
`smarts_get_pipeline` before trying to upload a genome.

## Public datasets

Read archives hold the raw data behind published papers, which is useful for
benchmarking a pipeline or reproducing a result. Two things to set expectations
on:

- **Accessions are hierarchical** — study, sample, experiment, run. A paper
  usually cites the study; the runs beneath it are the actual files
- **They are large.** A single human RNA-seq run is multiple gigabytes, and
  downloading a study is a serious undertaking, not an incidental step

## Where to go next

- A gene record, now what it does → `protein-function`
- Papers behind a dataset → `literature-and-evidence`

## Honesty

Accessions, coordinates and assembly names must come from a tool result.
**Never recall an accession or a genomic coordinate from memory** — they are
exactly the kind of fact that comes out fluent and wrong, and a wrong accession
sends the user to someone else's data. If the smarts.bio tools are unavailable
or unauthorized, stop and ask the user to connect their account.
