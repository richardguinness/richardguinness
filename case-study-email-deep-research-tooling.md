# Case study: Email Deep Research Tooling

> Draft case study for a private/anonymised project. Keep dataset names, client details, infrastructure hostnames, screenshots, paths, and sensitive examples out of any public version.

## Summary

Email Deep Research Tooling is a privacy-focused research system for working with large email archives. It turns an MBOX archive into a structured, searchable evidence base with keyword search, semantic retrieval, reranking, adjacent-context expansion, and cited synthesis.

The design goal is deliberately not “chat with your inbox”. The aim is a traceable research workflow: move from a broad question to a ranked evidence set that a human can inspect, challenge, refine, and cite.

## What problem it solves

Email archives are messy research corpora:

- messages contain inconsistent headers, quoted replies, HTML/plaintext variants, labels, attachments, and duplicate context;
- relevant evidence is often split across conversations or buried in attached documents;
- simple keyword search misses conceptual matches;
- semantic search can retrieve plausible but incomplete snippets;
- generated summaries are only useful if the underlying evidence can be audited.

This project builds a pipeline and research workflow around those constraints.

## Architecture

```text
EMAIL DEEP RESEARCH TOOLING PIPELINE

PIPELINE                                                      STACK / COMPONENTS
MBOX archive
(emails + attachments)
        │
        ▼
┌────────────────────────────────────────────────────────────┐
│ 1. Ingest + normalise                                      │   • Python mailbox
│                                                            │   • psycopg2
│   parse messages        save attachment files              │   • inscriptis
│   headers / bodies      capture attachment metadata        │   • mail-parser-reply
│          └──────────────┬──────────────┘                   │   • transformers / torch
│                         ▼                                  │
│   markdown body, latest reply, spam/ham probabilities      │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ 2. Store for auditability                                  │   • Postgres
│                                                            │   • SQL schemas / views
│   raw → clean → model → access layers                      │   • psycopg2 / asyncpg
│   stable IDs, headers, labels, timestamps, attachments     │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ 3. Build retrieval index                                   │   • MarkItDown
│                                                            │   • HF tokenizers
│   convert attachments to Markdown                          │   • sentence-transformers
│   concatenate body + attachments with explicit tags        │   • FastAPI model service
│   chunk by token budget, preserving body/attachment type   │   • Weaviate BYO vectors
│   embed chunks and ingest vectors + metadata               │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ 4. Retrieve + rerank evidence                              │   • Weaviate
│                                                            │   • BM25 / hybrid / vector
│   semantic search       keyword search                     │   • embeddings service
│          └──────────────┬──────────────┘                   │   • CrossEncoder reranker
│                         ▼                                  │   • neighbour expansion
│   rerank candidates, expand adjacent chunks, dedupe        │
└─────────────────────────┬──────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────┐
│ 5. Research workflow                                       │   • PydanticAI
│                                                            │   • privately hosted models
│   ranked evidence list: document IDs, scores, context      │   • FastAPI / Slack Bolt
│                         │                                  │   • Logfire / Phoenix
│                         ▼                                  │   • marimo notebooks
│   human review: inspect, challenge, refine                 │
│        ├─ synthesis with traceable references              │
│        └─ feedback: refine query / expand search           │
└────────────────────────────────────────────────────────────┘
```

## Pipeline walkthrough

### 1. Ingest and normalise

The ingestion pipeline reads an MBOX archive, extracts message headers and body variants, saves attachments to disk, and records stable metadata in the database.

Derived enrichment steps include:

- HTML/plaintext conversion to Markdown;
- latest-reply extraction to reduce quoted-thread noise;
- spam/ham scoring;
- attachment metadata capture.

### 2. Store for auditability

The database is deliberately layered:

- `raw` preserves extracted source data;
- `clean` materialises normalised/enriched records;
- `model` stores derived artefacts used for retrieval;
- `access` exposes stable query surfaces for tools and services.

This makes the pipeline inspectable and repeatable, and creates clear places to validate or rebuild derived state.

### 3. Build the retrieval index

Email body text and successfully processed attachment text are concatenated into a structured document format with explicit tags, for example:

```text
<email_body>
...
</email_body>

<attachment id="..." filename="..." mime="...">
...
</attachment>
```

That combined document is then chunked with token-budget awareness while preserving body/attachment provenance. Embeddings are generated by a privately hosted embeddings service and supplied to Weaviate as bring-your-own vectors, alongside metadata such as email ID, chunk index, chunk type, sender, subject, and timestamps.

### 4. Retrieve and rerank evidence

The retrieval workflow combines multiple strategies:

- vector search for conceptual matches;
- BM25 keyword search for exact names, acronyms, and identifiers;
- hybrid search where both are useful;
- reranking using a privately hosted CrossEncoder-style service;
- adjacent chunk expansion to recover surrounding context from multi-chunk emails;
- deduplication and score-based candidate selection.

This avoids relying on a single retrieval mode and makes it easier to explain why a document was selected.

### 5. Research workflow and synthesis

The research layer turns a user question into facets/sub-queries, collects and reranks candidates, expands context, and produces a ranked evidence list. Optional synthesis packs selected passages into a model prompt and produces a cited summary.

The important design constraint is that the generated answer is secondary to the evidence trail. The output should make it clear:

- what was searched;
- how many candidates were considered;
- which passages were selected;
- where claims in the summary came from;
- what gaps or limitations remain.

## Privacy and local-model focus

A major motivation was keeping sensitive archive data under private control. The system supports privately hosted model services for embeddings, reranking, and local/controlled inference, rather than assuming that archive content can be sent to a third-party SaaS model endpoint.

This shaped the design:

- model services are separated behind simple HTTP APIs;
- embeddings and reranking can run on local compute;
- the database and vector store are deployed on controlled infrastructure;
- evidence and logs can be inspected without exposing corpus content publicly.

## Engineering highlights

- End-to-end data pipeline from raw archive to searchable evidence base.
- Explicit database layering for auditability and rebuilds.
- Attachment-aware document construction and token-aware chunking.
- Privately hosted embedding and reranking services.
- Hybrid retrieval strategy: vector, BM25, filters, reranking, neighbour expansion.
- Cited synthesis with human review rather than untraceable answer generation.
- Deployment-oriented structure with containerised services and environment-specific configuration.

## Stack

Python, Postgres, SQL, Weaviate, FastAPI, Docker, sentence-transformers, transformers, torch, MarkItDown, inscriptis, PydanticAI, privately hosted model services, tracing/observability tooling.

## What I would improve next

- Add a polished reviewer UI for evidence triage.
- Add more systematic retrieval evaluation sets.
- Improve provenance display for attachments and multi-chunk emails.
- Expand run-level reporting so every research result includes coverage statistics and limitations.
- Package a sanitised demo corpus for public demonstration.
