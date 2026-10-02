---
name: small-molecules
description: Work with small molecules and drugs on smarts.bio. Use when the user has a SMILES string, SDF or MOL file, asks about a compound's drug-likeness or ADMET properties, wants bioactivity data for a molecule or target, or asks about an approved drug.
---

# Small molecules

Viewing compounds, computing their properties, and looking up what's known about
them. A SMILES string needs no upload beyond saving it as a file.

## Getting a molecule in

Formats that work: SMILES (`.smi`), InChI, MDL molfile (`.mol`), SDF (`.sdf`),
MOL2, XYZ.

A SMILES string from the conversation goes in with `smarts_upload_file` as a
`.smi` file.

**SMILES and InChI carry no 3D coordinates**, so the viewer resolves them to a
3D structure through an external service before it can draw anything. That means
a `.smi` file needs a network round trip the other formats don't, and an invalid
SMILES fails there rather than at parse time. If a molecule won't render, check
the SMILES before anything else.

## Viewing

`smarts_view_file` opens molecules in an interactive 3D viewer with selectable
representation and colouring. For a structure with a bound ligand, the structure
viewer is usually the better view; see `protein-structure`.

## Properties — two different tools, and the difference matters

**`smarts_analyze_file` gives Lipinski only.** Formula, atom and bond counts,
molecular weight, LogP, H-bond donors and acceptors, rotatable bonds, violation
count, and a drug-likeness interpretation. Fast, and enough for a first look.

**Real ADMET is a separate tool: `admet-predictor`.** It adds the predictions
`analyze_file` does not have — aqueous solubility, Caco-2 permeability, human
intestinal absorption, blood-brain barrier penetration, CYP3A4/2D6/2C9 inhibition
and hERG cardiotoxicity flags — plus a composite score it ranks a library by. It
takes a SMILES string or a library file, runs 5–20 minutes on ECS, and returns
ranked JSON and CSV. Find it with `smarts_list_tools`, check its parameters with
`smarts_get_tool`, run it with `smarts_run_tool`.

So: if the user asks whether a compound is drug-like, `analyze_file` answers. If
they ask about absorption, brain penetration, metabolism or cardiotoxicity, that
is `admet-predictor` — do not answer those from a Lipinski result.

**These are predictions from models, not measurements.** Say so. They are useful
for triage and ranking, not for deciding that a compound is safe or orally
available.

Lipinski in particular is a rule of thumb with well-known exceptions — many
approved drugs violate it. Report violations as a flag worth looking at, not as
a verdict.

## Known data

For what's actually been measured, call `smarts_list_tools` and search. The
platform reaches bioactivity databases and regulatory drug data. **Never invent
a tool id.**

| The question | Source |
|---|---|
| Has this been tested, against what, how potently | Bioactivity databases (ChEMBL) |
| What compounds hit this target | Bioactivity databases, by target |
| Is there an approved drug, what's on its label | Regulatory drug data (FDA) |

Measured bioactivity beats predicted properties every time. When both exist,
lead with the measurement.

## Finding new compounds against a target

Virtual screening is a pipeline, not a lookup: `structure-based-drug-discovery`
takes a target structure and a ligand library and screens them. See
`protein-design`. The blocker is usually the ligand library — if the user
doesn't have one in the workspace, solve that first.

For narrower questions the tools are faster than the pipeline:

- **`molecular-docking`** — pose and score specific ligands against a specific
  pocket, rather than screening a library
- **`molecular-dynamics`** — simulate a complex over time, for stability of a
  pose rather than whether one exists
- **`molecule-variant-generator`** — enumerate analogues around a hit, which
  feeds straight into `admet-predictor` for ranking

That last pair is the useful loop for lead optimisation: generate variants, score
them for ADMET, take the top few forward.

## Honesty

Predicted properties are predictions; measured bioactivity is measured. Keep the
two visibly apart in anything you report.

**Never state a compound's properties, targets or approval status from memory.**
Drug facts recalled from training are fluent and frequently subtly wrong — a
wrong potency or a wrong indication is indistinguishable from a right one in a
summary. If the smarts.bio tools are unavailable or unauthorized, stop and ask
the user to connect their account.

Nothing here is medical or safety advice about taking a compound.
