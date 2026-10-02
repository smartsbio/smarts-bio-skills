---
name: protein-function
description: Find out what a protein does using smarts.bio's annotation databases. Use when the user asks what a protein or gene does, wants domains, GO terms, pathways, or protein-protein interactions, has an unknown protein sequence, or wants to know what a set of genes has in common.
---

# Protein function and pathways

What a protein *is* — its domains, its annotations, what it interacts with, what
pathways it sits in. This is database work, not computation, and most of it
needs no file upload.

## Find the tools first

Call `smarts_list_tools`, then `smarts_get_tool` for the one you pick.
**Never invent a tool id.**

| The question | Source |
|---|---|
| What is this protein, canonically | `uniprot-toolkit` |
| What domains does it contain | `interpro-toolkit` |
| What is it annotated as doing | `go-toolkit` |
| What does it interact with | `string-db` |
| How are gene, protein, disease, pathway and drug related | `biograph` |
| What does this gene look like at the locus | `ensembl`, `ncbi-gene` |

Two routes people miss:

**Pathways do not have their own tool.** There is no KEGG tool in the catalogue.
KEGG pathways come back from `smarts_analyze_file` on a **protein sequence** —
the protein analysis emits domains, UniProt records, pathways and interactions
together. So "what pathways is this in" is answered by analysing the sequence, or
by `biograph`, not by a pathway lookup.

**`biograph` is the knowledge graph** over genes, proteins, diseases, variants,
pathways and drugs *with their relationships*. When the question spans two of
those — "which drugs target proteins in this pathway" — it is one query instead of
four lookups and a manual join. It's the most under-used tool in the catalogue.

**For an unknown protein with no useful annotation**, `protein-function-annotation`
predicts GO terms and an EC class from structure and sequence with DeepFRI, and
names the residues driving each prediction. 1–3 minutes. Label it a prediction,
clearly separate from a curated annotation.

## Identify the protein before searching

Most failures here are identity failures, not database failures.

- **An accession** (`P04637`) is unambiguous — best input
- **A gene symbol** needs an organism. `TP53` in human and `Trp53` in mouse are
  different proteins with different annotations
- **A sequence with no name** — run a homology search first to find out what it
  is, then annotate. See `sequence-analysis`

If a well-studied protein returns almost nothing, the identifier is wrong before
the database is. Try the synonym.

## Build an answer, not a dump

A protein record is long and most of it is irrelevant to the question asked.
Lead with what the protein does in one or two sentences, then support it with
the domains that explain the function, the pathways it participates in, and the
interactions that matter.

**Annotations carry evidence codes, and they are not equal.** Experimentally
determined annotations are worth more than ones inferred from electronic
similarity. When an annotation is doing real work in your answer, say where it
came from. "Inferred from electronic annotation" is a weaker claim than the
same sentence without that caveat.

## Sets of genes

"What do these genes have in common?" is enrichment, not lookup. Annotate the
set, then look for shared terms, pathways and interaction clusters. Report what
is actually shared across the set rather than what is interesting about
individual members — the latter is the easy answer and the wrong one.

## Where to go next

- Structure of the protein → `protein-structure`
- Design or engineer it → `protein-design`
- What's published about it → `literature-and-evidence`
- A variant in it → `variant-interpretation`
- Where it's expressed → `transcriptomics`

## Honesty

Every annotation, domain, pathway and interaction must come from a tool result,
with the source named. **Do not describe a protein's function from memory.** It
is the single easiest failure here: recalled protein biology is fluent, often
roughly right, and completely indistinguishable from a database lookup — so the
user has no way to know which they got. If the smarts.bio tools are unavailable
or unauthorized, stop and ask the user to connect their account.
