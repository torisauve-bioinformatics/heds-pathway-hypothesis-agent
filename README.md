# Pathway Hypothesis Agent
An AI agent that generates an initial gene pathway for disease pathogenesis based on enriched pathway context. Hypotheses are validated against StringDB, KEGG, and Open Targets evidence. 
Data from these sources are gathered via API calls to the tools, which are then transformed into pandas dataframes for consistent downstream handling. 

## How it works
1. **Input**: a list of candidate genes plus the enrichment context that produced them, and a target disease.
2. **Evidence gathering**: the pipeline queries StringDB (protein-protein interactions), KEGG (pathway membership), and Open Targets (gene-disease association score) for each gene.
3. **Hypothesis generation**: this evidence, along with enrichment context, is assembled into a prompt and sent to Claude, which proposes a candidate mechanistic pathway connecting the genes to the disease.


## Setup 
1. Clone this repository and install dependencies:
```bash
pip install -r requirements.txt
```
2. Get an Anthropic key from [console.anthropic.com](https://console.anthropic.com) (Settings → API keys).
3. Set your API key and workspace ID as environment variables:
```bash
export ANTHROPIC_API_KEY="sk-ant-api03-your-actual-key-here"
export ANTHROPIC_WORKSPACE_ID="your-workspace-id-here"
```
4. Launch Jupyter from the same terminal session so it inherits the environment variable:
```bash
jupyter notebook
```

## Usage
```python
test_genes = ["APOC1", "RDH10", "APOE", "BMP6", "CES1", "RAB38", "C3"]

evidence = gather_evidence(
    test_genes,
    "hypermobile Ehlers-Danlos syndrome",
    "GO:0046890 regulation of lipid biosynthetic process (padj=0.038)"
)

prompt = build_claude_prompt(
    disease_name=evidence["disease_name"],
    candidate_genes=evidence["candidate_genes"],
    enrichment_context=evidence["enrichment_context"],
    string_df=evidence["string_df"],
    kegg_df=evidence["kegg_df"],
    opentargets_scores=evidence["opentargets_scores"]
)

hypothesis = call_claude(prompt)
print(hypothesis)
```
## Limitations & future work 
- **Proof of concept, not a validated research tool.** Outputs are hypotheses for researcher review, not confirmed findings.
- **Absence of evidence is not evidence of absence.** A 'None' score from Open Targets or no KEGG pathway match means no "curated* association currently exists. It does not mean that no biological relationship exists. This is expected and relevant for diseases like hEDS with no confirmed genetic cause.
- **Gene symbol resolution can be ambiguous** across KEGG, StringDB, and Ensembl/Open Targets IDs; mismatches are logged as warnings rather than failing silently.
- **Planned extensions**:
  - **Literature context via Pubmed.** Surface relevant abstracts for a proposed hypothesis, kept deliberately unscored as literature co-occurrence in an abstract does not reliably indicate mechanistic relevance the way structured database evidence does
  - **Automated revision loop,** feeding verification back to Claude for hypothesis refinement across multiple iterations.
  - **Reactome integration** for reaction-level mechanistic detail beyond KEGG's pathway-membership check.
  - **CLI wrapper** for repeatable command-line runs.

## Tech stack 
Python, pandas, 'requests', Anthropic API (Claude), StringDB API, KEGG Rest API, Open Targets GraphQL API

