---
name: protein-structure
description: Work with 3D protein structures on smarts.bio. Use when the user has a PDB or mmCIF file, asks to see a structure, wants secondary structure composition or binding sites, asks whether a structure exists for a protein, or mentions AlphaFold.
---

# Protein structure

Finding a structure, showing it, and analysing it. The viewer is interactive, so
the user can rotate and inspect rather than read a description.

## Find a structure if they don't have one

Two sources, and the difference matters:

- **Experimental structures** — crystallography, cryo-EM, NMR. Real
  measurements, with a resolution
- **Predicted structures** (AlphaFold) — models, with per-residue confidence

Call `smarts_list_tools` to see what's available for structure lookup and run it
with `smarts_run_tool`. **Never invent a PDB ID or UniProt accession** — look it
up or ask. A plausible-looking four-character code that belongs to a different
protein produces an entire analysis of the wrong molecule.

Prefer experimental where it exists and covers the region of interest. Fall back
to predicted otherwise, and say which one you used — it changes how much weight
the result carries.

**Predicted structures carry confidence scores, and low-confidence regions are
usually disordered rather than wrong.** Don't interpret a floppy loop in a
prediction as a structural finding.

## Show it

`smarts_view_file` opens PDB and mmCIF in an interactive 3D viewer — the user
can rotate it, change representation and colour scheme. This is far better than
describing it.

Get the file into the workspace first. A structure you fetched from a database is
a tool result, not a file: write it with `smarts_upload_file` before viewing it.

Large structures are read from the start of the file, so a very large complex may
be missing some atoms in the view. If the user is reasoning about completeness —
chain counts, whether a subunit is present — mention it.

## Analyse it

`smarts_analyze_file` on a structure returns four things and an interpretation:

- **Structure info** — the chains with their residue counts, total residues and
  atoms, the resolution, and the experimental method (X-ray, NMR, cryo-EM)
- **Secondary structure** — helix, sheet and coil residue counts and percentages
- **B-factor** — mean, standard deviation and range
- **Interpretation**, including a fold assessment

Two of those are more useful than they look:

- **The chain list tells you what you actually have.** A PDB entry often contains
  more than the protein of interest — other subunits, antibodies used for
  crystallisation, tags. Check it before reasoning about the molecule
- **For an AlphaFold model, the B-factor column is pLDDT.** So the B-factor
  statistics *are* the confidence read-out: a low mean means a largely
  low-confidence model. For an experimental structure the same numbers mean
  thermal motion and disorder instead. Say which one you are reading

**Resolution**, for experimental structures: below ~2.5 Å side-chain positions are
reliable; above ~3.5 Å they often are not, which matters for anything involving a
pocket.

**No binding-site or pocket detection.** The analysis does not identify binding
sites, and it does not extract ligands or cofactors — so don't report either from
it. Ligands are visible to the user in the 3D viewer, and for a real pocket
question the routes are `structure-based-drug-discovery` (binding site
identification as part of the pipeline) or the `molecular-docking` tool.

## Structure-level tools worth knowing

Beyond viewing and analysing, found via `smarts_list_tools`:

- **`protein-complex-prediction`** — predict how two chains assemble, when the
  question is about an interface rather than one molecule
- **`protein-stability-prediction`** — thermostability of a sequence or design
- **`protein-variant-scoring`** — the effect of a specific mutation, which is
  usually what "will this substitution break it" really means
- **`molecular-docking`** — a specific ligand into a specific pocket

## Where to go next

- What the protein does → `protein-function`
- Design a binder or engineer it → `protein-design`, which takes a structure as
  its best input
- Screen small molecules against it → `protein-design`, structure-based drug
  discovery
- A variant that falls in this structure → `variant-interpretation`

## Honesty

Report what the analysis computed and what the file contains. **Do not describe
a structure's fold, binding site or ligands from memory** — a recalled
description of a well-known protein is fluent and unverifiable. If a structure
couldn't be found, say so rather than describing what it probably looks like. If
the smarts.bio tools are unavailable or unauthorized, stop and ask the user to
connect their account.
