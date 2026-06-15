# Piper

**Piper** is an AI-powered research platform combining two systems: a **multi-agent extraction pipeline** that automatically extracts structured reaction and conditions data from chemistry publications, and a **knowledge-graph-augmented RAG research assistant** for scientific literature.

> **Note:** The full codebase will be made publicly available shortly after the associated publication. This repository serves as a preview of the system architecture and capabilities.

---

## Table of Contents

1. [Overview](#overview)
2. [Piper Multi-Agent — Structured Data Extraction](#piper-multi-agent--structured-data-extraction)
3. [Piper RAG — Research Assistant](#piper-rag--research-assistant)
4. [Installation](#installation)
5. [Configuration](#configuration)
6. [Running the Application](#running-the-application)
7. [Output Format](#output-format)

---

## Overview

| Component | What it does |
|---|---|
| **Piper Multi-Agent** | Automated extraction of reaction conditions, SMILES, catalysts, and substrates from PDF chemistry articles |
| **Piper RAG** | Answers research questions across your literature library using hybrid semantic and knowledge-graph retrieval with LLM synthesis |

Both components share a unified Streamlit interface and are configurable through `context.json`.

---

## Piper Multi-Agent — Structured Data Extraction

Piper Multi-Agent processes a chemistry PDF end-to-end and produces structured JSON containing every reaction entry reported in the paper — including SMILES strings, catalysts, solvents, yields, temperatures, and R-group substitution data.

### What it extracts

- Reactant and product SMILES (OCSR-verified)
- Catalyst / photocatalyst identity and loading
- Reaction conditions: solvent, temperature, time, light source, atmosphere
- Yield per entry
- R-group substitution tables (scaffold + variant SMILES per row)
- Scope tables (substrate × condition grids)

### How it works

The pipeline takes a PDF as input and coordinates a team of specialised agents — each responsible for a single task — to parse, classify, resolve, and extract chemistry data. Cheap deterministic tools (optical chemical structure recognition, PubChem) run first; LLM agents handle only the cases those tools cannot resolve. This keeps costs low while maintaining accuracy across a wide range of paper formats.

The structured output also feeds a chemistry **knowledge graph** of reactions, compounds, conditions, and source papers, which the research assistant draws on for retrieval.

---

## Piper RAG — Research Assistant

A retrieval-augmented generation assistant connected to your literature library, combining semantic search with a chemistry knowledge graph.

### Features

- **Multimodal embeddings** — text, figures, and documents represented in a single vector space
- **Knowledge-graph retrieval** — reactions, compounds, conditions, and papers linked as a graph for relationship-aware search
- **Hybrid retrieval with fusion** — combines semantic (vector) search and graph-based search, merging the two ranked lists into one
- **Semantic chunking** — heading-based splits; tables kept atomic
- **Multi-path query engine** — routes to the right retrieval strategy per query type
- **Dynamic retrieval** — adjusts retrieved context based on relevance drop-off
- **Summarize-then-synthesize** — per-chunk summaries merged into one final answer

### Query types

| Type | Example |
|---|---|
| `general_query` | "What is the current state of photoredox catalysis?" |
| `filtered_query` | "What did Smith et al. report about Ir catalysts in 2022?" |
| `list_query` | "List all papers on nickel catalysis published after 2020" |
| `structured_extraction_query` | "What yields were reported for C–N coupling reactions?" |
| `relationship_query` | "Which catalysts have been used with K2CO3 as base?" |

---

## Installation

**Prerequisites:** Python 3.10+, Git

```bash
git clone https://github.com/lebertbill/Piper.git
cd Piper

python -m venv venv
source venv/bin/activate          # Windows: venv\Scripts\activate

pip install -r requirements.txt
pip install openai
```

---

## Configuration

Copy the template and fill in your paths and API keys:

```bash
cp context_template.json context.json
```

**Minimum required fields in `context.json`:**

```json
{
  "openrouter_api_key": "your_openrouter_api_key",
  "folder_path": "/path/to/document/library",
  "model": {
    "default": "openai/gpt-4o-mini"
  }
}
```

All model names, feature toggles, and extraction settings are configurable through the UI or directly in `context.json`.

---

## Running the Application

### Streamlit UI (recommended)

```bash
streamlit run app.py
```

The UI provides:
- **Extraction tab** — upload a PDF, configure models, run extraction, inspect results
- **Piper RAG tab** — ask questions across your literature library
- **Token tracker** — live cost and token usage breakdown

### Command line (RAG only)

```bash
python main.py
python main.py --query "What solvents were used in Ni-catalysed cross-coupling?"
```

---

## Output Format

Extraction results are written to `extracted_data/<paper_name>/<run>/processed/extraction_results.json`:

```json
{
  "table_name": "Table 1",
  "image_path": "..._artifacts/image_000009_abc.png",
  "specific_reactions": [
    {
      "entry_id": "1",
      "reactants": [
        { "label": "1a", "smiles": "c1ccccc1Br", "name": "bromobenzene" }
      ],
      "products": [
        { "label": "2a", "smiles": "c1ccc(-c2ccccc2)cc1", "name": "biphenyl" }
      ],
      "conditions": [
        { "role": "catalyst",  "text": "[Ir(ppy)3]", "quantity": "2 mol%" },
        { "role": "solvent",   "text": "MeCN",        "quantity": null },
        { "role": "yield",     "text": "85%",         "quantity": null }
      ]
    }
  ]
}
```

---

## Citation

> Publication details will be added upon release.

---

## License

License information will be provided alongside the full code release.
