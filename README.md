# smarts.bio

Run real bioinformatics on your own data, from a conversation with Claude.

smarts.bio is a bioinformatics platform: sequence analysis, variant calling,
RNA-seq, ATAC-seq and ChIP-seq pipelines, AI protein design, interactive file
viewers, and live search across literature, preprints, patents and clinical
trials. This plugin bundles the smarts.bio connector with skills that teach
Claude the judgement behind it — which pipeline fits a goal, which database
answers a question, what a result supports, and where the platform stops.

## Use it

Connect your smarts.bio account from the plugin's **Connectors** tab, then
describe what you want in plain English:

- *"What can smarts.bio do?"* — walks you through a first analysis
- *"Analyse this sequence"* — paste it straight into the chat, no upload needed
- *"I have FASTQ files on my laptop"* — you get a private upload link
- *"Align these reads and call variants"* — runs the pipelines in order and
  follows them to completion
- *"Design a nanobody against this antigen"* — AI protein design with
  computational validation
- *"What's known about this gene?"* — published papers, preprints, patents and
  trials together

Long-running jobs keep working after you close the conversation. Your files stay
in your own workspace.

## What's in it

Thirteen domain skills. Each one carries the scientific judgement for its field —
which pipeline fits a goal, which database answers a question, what a result does
and doesn't support, and what the platform cannot do.

Orientation and mechanics are deliberately **not** skills. Getting data in,
running a pipeline, picking a viewer and finding a tool are all delivered by the
smarts.bio connector itself, in its server instructions and through
`smarts_get_started` — so they reach every host, not only the ones that load
skills, and there is one copy to keep correct.

| Skill | For |
|---|---|
| `variant-calling` | GATK, WES, WGS, germline vs somatic |
| `variant-interpretation` | ClinVar, ClinGen, dbSNP, GWAS catalogue |
| `sequence-analysis` | GC, ORFs, CpG, restriction sites, codon usage, BLAST |
| `transcriptomics` | RNA-seq pipeline, tissue expression, count-matrix statistics |
| `epigenomics` | ATAC-seq, ChIP-seq, peak calling, motif enrichment, interval tools |
| `protein-function` | UniProt, InterPro, GO, STRING, pathways, knowledge graph |
| `protein-structure` | PDB/mmCIF, AlphaFold, chains, secondary structure, pLDDT |
| `protein-design` | Binders, nanobodies, enzyme engineering, virtual screening |
| `small-molecules` | SMILES/SDF, Lipinski, ADMET, docking, bioactivity |
| `metagenomics` | 16S vs shotgun, abundance-table statistics, and what is missing |
| `literature-and-evidence` | Papers, preprints, patents, trials, drug data |
| `public-data` | Reference genomes, annotations, SRA, taxonomy |
| `data-wrangling` | Format conversion, table statistics, archives |

Every skill instructs Claude to report only what a tool actually returned, and
to stop rather than answer from memory when the connector isn't available. Each
one also states what the platform *cannot* do in its area, because a confident
answer that no tool produced is the failure mode that users cannot detect.

## Data

The plugin itself stores and sends nothing. Its skills direct Claude to the
smarts.bio MCP server at `mcp.smarts.bio`, which you authorize with your own
smarts.bio account over OAuth.

Through that connector, content you ask Claude to analyse — file contents,
sequences, and search terms — is sent to your smarts.bio workspace and processed
there. Files you upload stay in your workspace. Searches reach the public
databases behind them, such as PubMed, patent offices and clinical trial
registries.

Without an authorized connector the skills do nothing: they instruct Claude to
stop and ask you to connect rather than answering from memory.

## Links

- Platform: https://smarts.bio
- Documentation: https://smarts.bio/docs/mcp
