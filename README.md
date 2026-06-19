> [!NOTE]
> In this repo, you'll find examples of projects I've worked on mostly for personal interest, with a few exceptions.


| Project | Quick overview | Main points of interest | Links |
| --- | --- | --- | --- |
| 🕵️ **Email Deep Research Tooling** | Privacy-focused research tooling over large email archives. | MBOX ingestion; attachment extraction; Postgres audit layers; privately hosted embeddings/reranking; Weaviate retrieval; BM25/vector/hybrid search; neighbour expansion; cited synthesis; human review loop. | [Case study draft](./case-study-email-deep-research-tooling.md) |
| 🦀 **WASM MNIST Trainer** | Browser-based neural-network trainer with Rust compiled to WebAssembly. | Rust numerical core; `wasm-bindgen`; Web Worker training; configurable MNIST training UI; metrics; model import/export; canvas digit prediction. | ✨ [Live demo](https://richardguinness.github.io/wasm-mnist-trainer/) · 🚀 [Public repo](https://github.com/richardguinness/wasm-mnist-trainer) |
| 🏛️ **Historic-property planning analytics** | Public-data pipeline for identifying historic properties with planning-activity signals. | Data ingestion; enrichment; entity matching; evidence modelling; SQLite tracking; generated review packages; analyst-in-the-loop workflow. | Case study coming soon |
| 🧠 **Mindlist** | Lightweight task app with list and mind-map views. | Vanilla JS; no build step; shared state; nested tasks; SVG pan/zoom/drag; keyboard affordances; local persistence. | ✨ [Live demo](https://richardguinness.github.io/mindlist/) |
| 🧱 **Local infrastructure automation** | Private compute and service stack for data/AI projects. | IaC; virtualised Linux services; Docker stacks; internal DNS/reverse proxying; private model-service hosting; GitHub runners; repeatable environments. | Notes coming soon |

## Core stack

Python · SQL · Postgres/SQLite · FastAPI · Docker · Terraform · Weaviate · sentence-transformers · Rust · WebAssembly · JavaScript · GitHub Actions
