A lightweight, fund-detection-aware evaluation framework for benchmarking 
embedding models in a Retrieval-Augmented Generation (RAG) pipeline built 
over Islamic Fund Manager Reports (financial PDF data).

## Problem

Choosing the right embedding model for a RAG system is usually guesswork.
Raw vector similarity search across a large corpus of near-duplicate 
sections (e.g. repeated "Investment Committee" tables across 18+ funds) 
produces misleading results — the wrong fund's chunk often outranks the 
correct one purely due to boilerplate text similarity.

## What This Does

- Loads a frozen, chunk_id-stable corpus (`frozen_chunks.json`) generated
  from a real 100+ page financial PDF.
- Reuses the production chatbot's own fund-detection logic 
  (`FUND_NAMES`, alias resolution, fuzzy matching) to narrow the 
  candidate pool before running embedding similarity — mirroring how 
  the live RAG pipeline actually retrieves.
- **Auto-detects multi-chunk golden answers**: sections that exceeded 
  the chunking size limit and got split across multiple chunks are 
  discovered directly from chunk metadata (no manual guessing), and the 
  golden test set is expanded accordingly.
- Evaluates two embedding models side-by-side 
  (`sentence-transformers/all-MiniLM-L6-v2` vs `BAAI/bge-small-en-v1.5`) 
  across 40 real-world financial queries.
- Reports both **Recall (Coverage)** and **Precision** per question, 
  plus aggregate **Hit Rate**, **MRR**, and a combined **Composite Score**.
- Outputs a formula-driven Excel report (no hardcoded aggregates — every 
  average and verdict recalculates live in Excel).

## Metrics

| Metric | What it measures |
|---|---|
| Hit Rate | Did at least one expected chunk get retrieved? |
| MRR | How high did the correct chunk rank? |
| Recall / Coverage | What % of the full expected answer was retrieved? |
| Precision | Of the retrieved chunks, how many were actually relevant? |
| Composite Score | Weighted average of the above — used for final model selection |

## Result

On this corpus, `BAAI/bge-small-en-v1.5` outperformed 
`all-MiniLM-L6-v2` across every metric (94% vs 88% Hit Rate, 
0.43 vs 0.35 MRR) — with one notable exception: MiniLM was more 
reliable on performance/return-related queries, suggesting a 
field-type-aware hybrid retrieval strategy could be a next step.

## Tech Stack

Python · LangChain · HuggingFace Embeddings · NumPy (cosine similarity) 
· openpyxl (formula-driven Excel reporting)
