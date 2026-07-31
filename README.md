# Multi-stakeholder agents: does repeated search change a risk assessment, or just how much you trust it?

Four AI reviewers (Biology, Clinical, Regulatory, CMC/Manufacturing) assess a real historical drug trial decision, TRANSEURO, a Parkinson's cell therapy program, using only evidence available before the decision point. Four conditions test what changes when search is removed, capped at one round, or left unlimited.

**Read the full write-up here: [`case_study.md`](./case_study.md)**

## Repository contents

- `case_study.md`, the write-up: question, methodology, results, limitations
- `multistakeholder_agents.ipynb`, the full corpus-building and experiment pipeline
- `results_index.md`, maps every file in `Results/` to its condition and run
- `Results/`, every saved reviewer output and reasoning trace, unedited
- `transeuro_frozen_context.txt`, the exact evidence package every condition read from

## Running it yourself

```
pip install requests pandas anthropic python-dotenv
```

Create a `.env` file in this directory:

```
ANTHROPIC_API_KEY=your_key_here
NCBI_API_KEY=your_ncbi_key_here
NCBI_EMAIL=your_email@example.com
```

`NCBI_API_KEY` is optional (raises the PubMed rate limit, register free at ncbi.nlm.nih.gov/account), `NCBI_EMAIL` is NCBI's requested identifier, not a credential.

## A note on how this was built

The experimental design, case selection, the contamination protocol, the corpus methodology, the four-condition structure, and the evaluation approach are mine. The code implementation was built with Claude (Anthropic), including debugging the PubMed pipeline and the agent-calling logic. Where the notebook or write-up describes a design decision or a finding, that's my judgment; where it describes how a function works, that's collaborative implementation.
