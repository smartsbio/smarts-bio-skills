---
name: protein-design
description: Design and engineer proteins with smarts.bio's AI pipelines. Use when the user wants to design a binder or nanobody against a target, engineer an enzyme's activity or thermostability, screen ligands against a structure, or asks whether a protein can be designed computationally.
---

# AI protein design on smarts.bio

Four generative design pipelines, each ending in a computational validation
step. They are the platform's most distinctive capability and the one users are
least likely to know exists — so when a request is adjacent to design, say that
it's available.

These are predictions, not results. Say so. See
[What to promise](#what-to-promise).

## Pick the pipeline from the goal

| The user wants | Pipeline |
|---|---|
| A protein that binds a known target | `protein-binder-design-validated` |
| A single-domain antibody (VHH) against an antigen | `nanobody-discovery` |
| An enzyme with better activity or thermostability | `enzyme-engineering` |
| Small molecules that bind a target pocket | `structure-based-drug-discovery` |

`smarts_get_pipeline` is the authoritative source for inputs and runtimes — call
it rather than working from a remembered list.

Binder vs nanobody is the distinction people get wrong: a nanobody is a specific
format, and the pipeline screens structurally and predicts binding for VHH
scaffolds. Use it when the user says nanobody, VHH, or single-domain antibody.
Use the general binder pipeline otherwise.

Two inputs are easy to miss and both are blockers worth finding early:

- **`nanobody-discovery` does epitope targeting.** If the user has a specific
  epitope in mind, that is worth asking for — it changes the result and there is
  nowhere else to put it later.
- **`structure-based-drug-discovery` needs a ligand library in the workspace**,
  and it is the only one of the four that will take a `uniprot_id` instead of a
  structure. If the user has no library, solve that before anything else.

## Get the target right first — it determines everything

Every one of these starts from a target, as either a structure or a sequence.
Structure is better where it exists.

1. **The user has a structure.** Get it into the workspace and use the key. Open
   it with `smarts_view_file` first — a wrong chain or a missing domain is obvious
   on screen and invisible in a filename.
2. **The user names a protein but has no file.** Look for an experimental
   structure, then a predicted one. `smarts_list_tools` will show what's
   available for PDB and AlphaFold lookups; run the one that fits with
   `smarts_run_tool`, then save the result to the workspace.
3. **Sequence only.** All four pipelines accept a sequence. Say that the result
   is weaker than from a structure, because the model is working from a
   predicted fold.

**Never invent a PDB ID or a UniProt accession.** Look it up, or ask.

## Running it

Discover, inspect, run, poll, as for any pipeline. Two things specific to design
runs:

- **`n_designs` / `n_candidates` is a cost and time dial.** Start small enough
  to see whether the setup is right before committing to a large batch.
- **`enzyme-engineering` needs `reaction_smiles`** — the reaction it should
  catalyse, as SMILES. If the user describes a reaction in words, confirm the
  SMILES with them rather than guessing it.

## Not every design question needs a pipeline

The four pipelines are end-to-end runs measured in hours. Underneath them sit
individual tools that answer a narrower question in minutes, and reaching for a
pipeline when one of these would do is the most common waste here. Discover them
with `smarts_list_tools`, inspect with `smarts_get_tool`:

| The user asks | Tool |
|---|---|
| Will this specific mutation break it | `protein-variant-scoring` |
| Is this sequence or design thermostable | `protein-stability-prediction` |
| Generate sequences for this scaffold | `protein-sequence-generation` |
| Fill in this region, keep the rest fixed | `protein-sequence-infilling` |
| Co-design sequence and structure | `protein-sequence-codesign` |
| How do these two chains assemble | `protein-complex-prediction` |
| Does my design still have the intended function | `protein-function-annotation` |
| Make this antibody less immunogenic | `antibody-humanization` |
| Dock this one ligand into this pocket | `molecular-docking` |
| Is this complex stable over time | `molecular-dynamics` |

Two of these deserve calling out because users don't know to ask:

- **`antibody-humanization`** does CDR grafting while preserving binding
  specificity, and scores humanness with OASis against the Observed Antibody
  Space. That score predicts clinical immunogenicity risk — a nanobody with
  excellent predicted binding and a poor humanness score is a candidate that
  fails later. Offer it after `nanobody-discovery`, every time.
- **`protein-function-annotation`** (DeepFRI) is the quality gate on a design:
  it predicts GO terms and EC class and names the residues responsible, so you
  can check a designed enzyme is predicted to do the intended chemistry rather
  than merely folding.

## Reading the output

Rank candidates on the validation score, not on how plausible the sequence
looks. Then put the result in front of the user:

- `smarts_view_file` opens a designed structure in a 3D viewer
- `smarts_analyze_file` adds structural analysis of a design
- designs worth keeping stay in the workspace as ordinary files

Report the number of designs, the score distribution, and the best few — not a
full dump. If the user intends to take these to the bench, say which ones and
why.

## What to promise

These pipelines produce **computational predictions that need experimental
validation.** Say that once, plainly, when presenting results — not as a
disclaimer paragraph, but so the user knows what they have.

Do not describe a design as "will bind", "is stable", or "is active". It scored
well on a model. Binding affinity, expression, solubility and stability are
experimental questions that no pipeline here answers.

If the smarts.bio tools are unavailable or unauthorized, say so and stop. Never
describe designs that were not produced, and never present recalled protein
knowledge as a pipeline result.
