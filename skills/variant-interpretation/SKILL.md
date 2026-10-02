---
name: variant-interpretation
description: Work out what a genetic variant means using smarts.bio's clinical and population databases. Use when the user asks whether a variant is pathogenic or benign, gives an rsID or HGVS notation, asks about a mutation's clinical significance, or wants disease associations for a locus.
---

# Interpreting variants

This is a different job from calling variants. Calling asks "what changed";
interpretation asks "does it matter", and it is answered from curated databases,
not from a pipeline. No upload is needed if the user just has a variant ID.

**This is a research aid, not a clinical determination.** See
[What not to say](#what-not-to-say).

## Find the tools first

Call `smarts_list_tools`, then `smarts_get_tool` for the one you pick.
**Never invent a tool id.**

Match the source to the question:

| The question | Source |
|---|---|
| Has this variant been clinically classified | `ncbi-clinvar` |
| Is this gene genuinely linked to this disease | `clingen` |
| How common is it in the population | `ncbi-dbsnp` |
| Is this locus associated with a trait | `gwas-catalog` |
| How do this gene, variant, disease and drug connect | `biograph` |
| What does it do to the protein | see `protein-function` |
| Does this substitution destabilise the protein | `protein-variant-scoring` |

Two worth knowing about:

**`biograph`** is a knowledge graph over genes, proteins, diseases, variants,
pathways and drugs with their relationships. For "is there a drug that targets
this pathway" it replaces several lookups and a manual join.

**`protein-variant-scoring`** is a model, not an archive. It predicts the effect
of a substitution on the protein — useful for a variant with no clinical
classification, which is most of them. Keep it visibly separate from curated
evidence: a model score is not a classification, and must never be reported as
one.

## Get the variant identified properly first

Ambiguous input is the main failure here. Before searching, pin down:

- **An rsID** (`rs429358`) is unambiguous — use it directly
- **HGVS** (`NM_000546.6:c.215C>G`) needs its transcript; a different transcript
  gives a different coordinate for the same change
- **Genomic coordinates** need the build. GRCh37 and GRCh38 differ by millions
  of bases — a coordinate without a build is not a location

If the build or transcript is missing and matters, ask. Guessing produces a
confident answer about a different variant.

Working from a VCF in the workspace? Open it with `smarts_view_file` and pick
out the variants of interest rather than interpreting thousands.

## Read the classifications honestly

Clinical archives report submitted classifications, and submitters disagree.
When they do, say so — "three labs call it pathogenic, one calls it uncertain"
is the finding. Collapsing that into a single verdict destroys the information
the user needs.

**Variant of uncertain significance means uncertain.** Do not resolve a VUS into
a leaning. It is the most common classification and the most frequently
over-interpreted.

**Absence is not evidence of benignity.** A variant missing from an archive is
usually unstudied, not safe. Say "no classification found", never "no known
pathogenicity".

Population frequency is a strong signal in one direction only: common variants
are rarely highly penetrant, but rarity alone does not make a variant harmful.

## Build the picture

A useful answer usually combines: the clinical classification and who submitted
it, the population frequency, whether the gene–disease link itself is
established, and the predicted effect on the protein.

Then connect it to what the user has — if the variant came from their own VCF,
say where it sits among their other calls.

## What not to say

Do not diagnose, do not advise on medical decisions, and do not tell a user what
a result means for them or a family member personally. Point to a clinical
geneticist or genetic counsellor when the question turns personal.

Every classification must come from a tool result, with the source named. Do not
classify a variant from memory, and do not present recalled pathogenicity as a
database lookup — it is indistinguishable from a real one and the user cannot
check it. If the smarts.bio tools are unavailable or unauthorized, stop and ask
the user to connect.
