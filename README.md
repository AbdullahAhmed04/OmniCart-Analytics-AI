# OmniCart Analytics AI

**Enterprise Big Data & RAG Intelligence Engine** — a hybrid AI assistant that unifies structured retail transaction data with unstructured corporate policy documents, queryable through natural language.

## Overview

Traditional retail analytics setups isolate operational databases (sales, transactions, vendor performance) from unstructured knowledge (policy manuals, FAQs, vendor agreements). OmniCart Analytics AI bridges that gap with a dual-engine architecture:

- A **Structured Engine** (Apache Spark SQL) for querying transactional and vendor performance data
- An **Unstructured Engine** (Qdrant vector store + RAG) for querying corporate policies and FAQs
- A **LangChain ReAct Agent** (powered by Groq's Llama 3.3 70B) that automatically routes each question to the correct engine — or combines both when a question needs numeric data *and* policy context together

## Screenshot

![OmniCart Analytics AI Dashboard](dashboard_screenshot.png)

## Architecture

```
                    User Query (Gradio Chat UI)
                                │
                                ▼
                   LangChain ReAct Agent (Groq LLM)
                       /                    \
                      ▼                      ▼
         Spark SQL Query Tool        Policy Search Tool (RAG)
         ┌──────────────────┐        ┌──────────────────────┐
         │ sales_records     │        │ Qdrant in-memory      │
         │ vendor_performance│        │ vector store           │
         │ (Spark SQL views) │        │ (policy/FAQ chunks)    │
         └──────────────────┘        └──────────────────────┘
```

### Structured Engine
- **`sales_records`**: 5,000 synthetic e-commerce transactions across 5 Pakistani cities, 5 product categories
- **`vendor_performance`**: 10 vendors with order defect rates and tracking upload delays, used to demonstrate policy-rule violations (e.g. ODR > 1.5%)
- Registered as Spark SQL temporary views (`createOrReplaceTempView`), queried live by the agent via generated SQL

### Unstructured Engine (RAG)
- Corporate policy documents (shipping guidelines, product FAQs, vendor agreements) are chunked in **parallel using a PySpark Pandas UDF** wrapping LangChain's `RecursiveCharacterTextSplitter`
- Chunks are embedded using `sentence-transformers/all-mpnet-base-v2` (fully local, no API key required)
- Indexed into an **in-memory Qdrant vector store** for fast semantic similarity search

### Orchestration Layer
- A LangChain **ReAct agent** reasons about each question, decides which tool(s) to call, executes them, and synthesizes a final answer
- Powered by **Groq's free-tier API** running Llama 3.3 70B for fast, low-cost inference

### Interface
- A **Gradio chat dashboard** with example questions, live "currently using" engine indicator, and a custom red/white theme

## Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.10+ |
| Big Data Framework | Apache Spark (PySpark SQL) |
| LLM Orchestration | LangChain (ReAct Agent) |
| Vector Database | Qdrant (in-memory mode) |
| Embeddings | HuggingFace `sentence-transformers/all-mpnet-base-v2` |
| LLM Inference | Groq API (Llama 3.3 70B) |
| Interface | Gradio |
| Environment | Google Colab |

## Example Queries

| Question | Engine Used |
|---|---|
| "What is the total revenue generated in Lahore?" | Structured (Spark SQL) |
| "How long does standard shipping take?" | Unstructured (RAG) |
| "Why might Fast Fashion Lahore be at risk of suspension?" | Both — pulls defect rate from Spark SQL and the ODR policy rule from RAG |

## Running This Project

This project is designed to run entirely in **Google Colab**, completely free — no paid API keys, no cloud infrastructure, no local storage requirements.

1. Open `OmniCart_Analytics_AI.ipynb` in Google Colab
2. Get a free Groq API key at [console.groq.com](https://console.groq.com) → API Keys → Create API Key
3. In Colab, add it as a secret: 🔑 Secrets panel → Add new secret → name it `Groq_API_Keys` → paste your key → enable notebook access
4. Run all cells in order (Cell 1 → 1b → 2 → 3 → 4)
5. Cell 4 launches the Gradio dashboard with a public shareable link

No local dataset downloads are required — all data (transactions, vendor records, policy documents) is synthetically generated inside the notebook so the project is fully self-contained and reproducible.

## Project Structure

```
OmniCart-Analytics-AI/
├── OmniCart_Analytics_AI.ipynb   # Main notebook (all cells)
├── requirements.txt               # Python dependencies
├── README.md                      # This file
└── .gitignore
```

## Notes

- All data (transactions, vendor performance, policy documents) is **synthetically generated** — no external dataset dependency, no training required
- No models are trained from scratch; this project uses pre-trained embeddings + LLM inference with retrieval and tool-calling
- The Qdrant vector store runs in `:memory:` mode, so it resets each time the notebook restarts — this is intentional for a free, zero-infrastructure demo
