---
name: literature-and-evidence
description: Search and synthesize scientific evidence through smarts.bio. Use when the user asks what is known about a gene, protein, disease, drug or method, wants recent papers or preprints, asks whether something has been patented, or asks about clinical trials or approved drugs.
---

# Literature and evidence on smarts.bio

smarts.bio reaches published literature, preprints, patents, clinical trials and
drug approvals through live tools. This is the one capability that needs **no
file upload**, so it works for a user who has not got their data in yet — and
it's often the fastest way to show the platform doing something real.

Every claim here must come from a tool result. See
[Cite what you searched](#cite-what-you-searched).

## Find the right tools first

Call `smarts_list_tools` and read what's available before searching, then
`smarts_get_tool` for the one you pick. **Never invent a tool id.**

Match the source to the question:

| The user is asking | Search |
|---|---|
| What's established | `ncbi-pubmed` |
| What's new, last few months | `biorxiv-search`, `medrxiv-search`, `arxiv-search` |
| Metadata or a DOI across publishers | `crossref-search` |
| Is this protected / has someone claimed it | `uspto-patents-search` |
| Is anyone testing this in people | `clinical-trials-toolkit` |
| Is there an approved drug, what are its labels | `openfda-drug-toolkit` |
| How are these entities connected | `biograph` |
| Anything across EBI's databases at once | `ebi-search` |
| General context, non-academic sources | `tavily-search` |

Note that patent coverage is **USPTO**, so it is US filings. Don't present a
clear USPTO search as evidence that nobody anywhere has claimed something.

Most real questions need two or three of these. "Is anyone working on X?" is
answered badly by literature alone — a patent filing or a running trial is often
the more current signal.

## Search properly

**Start specific, then widen.** A gene symbol alone returns thousands of hits
and nothing useful. Add the biology: the disease, the process, the organism, the
method.

**Try the synonyms.** Gene and protein names change. If a search comes back
nearly empty on something that should be well studied, the name is usually
wrong before the database is.

**Preprints are not peer reviewed.** Label them as preprints when you cite them.
That distinction matters to the people using this.

## Synthesize, don't list

A list of titles is not an answer. Lead with what the evidence says, then
support it:

1. **The answer**, in two or three sentences
2. **The evidence**, grouped by what it shows rather than by source
3. **Where it disagrees** — conflicting results are a finding, not an
   inconvenience, and hiding them makes the summary useless
4. **What's missing** — if nothing addresses the question directly, that is the
   result, and worth saying plainly

Keep it to what was found. Twenty citations that each restate the same claim is
not more evidence than three.

## Connect it to the user's own data

This is where the platform is more than a search engine. When the user has data
in the workspace, close the loop:

- A variant they're looking at → what's published about that gene or locus
- A protein they analysed → known interactions, pathways, and the literature
- A design they produced → prior art in patents, and existing trials on the
  target

Offer that link explicitly. A user who came for a paper and leaves with their
own data analysed has understood what this is for.

## Cite what you searched

Every statement must trace to a tool result. Give enough for the user to find
the source — title, authors, year, and the identifier the tool returned.

**Do not fill gaps from memory.** If the searches found nothing, say the
searches found nothing. A recalled paper presented alongside real results is
indistinguishable from them, and is the one failure here that the user cannot
detect.

If the smarts.bio tools are unavailable or unauthorized, tell the user to
connect their account and stop. Do not answer the question from training data.
