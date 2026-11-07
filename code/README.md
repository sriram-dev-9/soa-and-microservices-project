# Cognix — Personal Knowledge Operating System with AI Memory

> **Hackathon Edition**: Built with **Google Gemini 3.8 Flash**, **LangGraph**, **FastAPI**, **Next.js (App Router + Tailwind CSS)**, **Local Embedded Qdrant**, **NetworkX Graph RAG**, and **SQLite Structured Memory**.

---

## 🌟 Overview & Architecture

Cognix implements an independently deployable microservices-oriented personal memory substrate based on the blueprint in the project specification:

```
[Input Sources] ➔ [Ingestion & Extraction] ➔ [Structured Memory Store (CCE Core)]
                                                  ├── 3.1 Semantic Search Index (Qdrant Embedded)
                                                  ├── 3.2 Concept / Relationship Graph (NetworkX)
                                                  └── 3.3 Structured Relational Memory (SQLite)
                                                                 ▲
                                                                 │
[User Query] ➔ [LangGraph Multi-Node State Machine] ─────────────┤
                    ├── 1. Query Understanding                   │
                    ├── 2. Hybrid Retrieval (Vector + Graph)     │
                    ├── 3. Multi-Hop Graph Reasoning             │
                    ├── 4. Relevance Scoring & Re-ranking        │
                    ├── 5. Adaptive Context Compaction           │
                    ├── 6. Gemini 3.8 Flash Generation           │
                    └── 7. Memory Update Feedback Loop ──────────┘
```

---

## 🚀 Key Features

1. **Explicit LangGraph Agent State Machine**:
   - Rather than relying on a fragile monolithic prompt, each phase (query parsing, vector + graph hybrid search, 2-hop graph reasoning, cross-candidate scoring, token budget compaction, generation, memory feedback) is an inspectable LangGraph node.
   - Real-time Server-Sent Events (SSE) streaming updates the UI node stepper in real-time.

2. **Concept Relationship Graph (Graph RAG)**:
   - Maintains an interconnected entity-relationship graph (`NetworkX`).
   - Enables **multi-hop traversal** to surface hidden, non-adjacent dependencies that naive vector search misses.
   - Interactive 2D Canvas force-directed visualizer with dragging, zooming, type-color badges, and 1st/2nd hop neighbor filtering.

3. **Structured Relational Ground-Truth (SQLite via SQLAlchemy)**:
   - Stores atomic facts/statements with confidence, verification status, categories, subject/predicate/object triples, and source provenance pointers.
   - Direct CRUD for transparency and user corrections.

4. **Zero-Friction Local Semantic Database**:
   - Uses embedded **Qdrant** (`QdrantClient(path="./data/qdrant_db")`), running 100% locally with cosine distance indexing and zero external Docker containers needed.

5. **3-Way Empirical Benchmark Arena (Section 7)**:
   - Direct head-to-head comparison between:
     1. **Cognix (Graph RAG + Adaptive Compaction)**
     2. **Flat Chunk RAG (Vector top-k only)**
     3. **Conversation History Replay (Naive context dump)**
   - Computes token efficiency (typically ~60-75% token reduction), retrieval latency, and multi-hop discovery paths.

6. **Source Provenance Drawer**:
   - Every stored fact or cited statement preserves an unforgeable provenance pointer back to the exact original document, note, or chat turn.

---

## 📦 Project Structure

```
soaproj/
├── backend/
│   ├── pyproject.toml              # Managed via uv
│   ├── run.py                      # Starts uvicorn server on port 8000
│   ├── app/
│   │   ├── main.py                 # FastAPI Gateway with CORS & auto-seed
│   │   ├── config.py               # Settings (Gemini model, storage paths)
│   │   ├── seed_data.py            # Rich demo dataset (ADRs, notes, roadmap)
│   │   ├── services/
│   │   │   ├── gemini_service.py   # Official google-genai SDK (gemini-3.8-flash, embeddings)
│   │   │   ├── vector_service.py   # Embedded Qdrant vector store
│   │   │   ├── graph_service.py    # NetworkX concept graph & multi-hop traversal
│   │   │   ├── structured_memory.py# SQLite facts, entities, events, provenance
│   │   │   ├── ingestion_service.py# Chunking, extraction & indexing pipeline
│   │   │   └── evaluation_service.py# 3-Way benchmark evaluation engine
│   │   ├── agent/
│   │   │   ├── state.py            # TypedDict AgentState
│   │   │   ├── nodes.py            # 7 distinct LangGraph state nodes
│   │   │   └── graph.py            # Compiled LangGraph StateGraph
│   │   └── routers/
│   │       ├── chat.py             # /api/v1/chat (SSE Streaming & LangGraph trace)
│   │       ├── ingest.py           # /api/v1/ingest (Document/chat indexing)
│   │       ├── memory.py           # /api/v1/memory/* (Search, graph, facts CRUD)
│   │       └── benchmark.py        # /api/v1/benchmark (Section 7 evaluation)
├── frontend/
│   ├── package.json
│   ├── next.config.ts              # Reverse proxy rewrite to FastAPI backend
│   └── src/
│       ├── app/
│       │   ├── page.tsx            # Main Studio layout with tabs
│       │   ├── layout.tsx
│       │   └── globals.css         # Dark cyberpunk/sleek glassmorphic theme
│       ├── components/
│       │   ├── Header.tsx          # System status, stats, Gemini API key modal
│       │   ├── ChatInterface.tsx   # SSE chat, markdown, citations, prompt chips
│       │   ├── LangGraphProgress.tsx # Live 7-step state machine stepper
│       │   ├── GraphVisualizer.tsx # 2D Force-Directed Canvas Graph Inspector
│       │   ├── MemoryTable.tsx     # Facts table, Entity catalog, Source archive
│       │   ├── BenchmarkView.tsx   # 3-Way head-to-head evaluation arena
│       │   └── ProvenanceDrawer.tsx# Raw document & provenance slide-over
│       └── lib/
│           ├── api.ts              # API client methods
│           └── types.ts            # TypeScript interfaces
└── README.md
```

---

## ⚡ Quick Start Guide

### 1. Requirements
- Python 3.11+
- `uv` (installed)
- Node.js 18+ and `npm`

### 2. Configure Gemini API Key
Create the local environment file from the included template, then paste your Gemini API key into `backend/.env`:
```bash
cp backend/.env.example backend/.env
```

Set the key in `backend/.env`:
```env
GEMINI_API_KEY=your_actual_gemini_api_key_here
GEMINI_MODEL=gemini-3.8-flash
GEMINI_EMBEDDING_MODEL=gemini-embedding-001
```
*Tip: You can also enter or update your Gemini key at any time directly in the web UI using the top-right **"Gemini Key"** button.*

### 3. Start Backend (FastAPI Microservices)
```bash
cd backend
uv run python run.py
```
Backend runs at: `http://127.0.0.1:8000`  
Interactive OpenAPI / Swagger Documentation: `http://127.0.0.1:8000/docs`

### 4. Start Frontend (Next.js)
```bash
cd frontend
npm run dev
```
Open your browser at: `http://localhost:3000`

---

## 🧪 Quick Hackathon Demo Walkthrough

1. **Explore the Substrate**:
   - The system automatically pre-seeds 4 comprehensive documents (ADR 001, Alex's preferences, Team Roadmap, Graph RAG technical analysis), producing ~30 graph nodes and 20+ verified relational facts!
2. **Chat & Agent Stepper**:
   - Go to the **"Chat & LangGraph Agent"** tab.
   - Click one of the suggested query chips, e.g.:
     `"What did we decide about Docker and how does it relate to our hackathon setup?"`
   - Watch the **LangGraph State Machine** stepper activate through all 7 nodes.
   - Inspect the generated answer and click on the `[#Source]` citation pill to open the original source in the **Provenance Drawer**.
3. **Inspect the Concept Graph**:
   - Switch to the **"Concept Graph Inspector"** tab.
   - Drag nodes, zoom in/out, or search for entities like `Docker`, `Alex`, `FastAPI`, `LangGraph`.
   - Click any node to highlight its 1st and 2nd degree relationships in the side inspector.
4. **Structured Facts & Add Memory**:
   - Go to the **"Structured Memory & Facts"** tab.
   - Add a custom fact or click **"Ingest Source"** to paste new notes.
5. **Run the 3-Way Benchmark Arena**:
   - Switch to the **"3-Way Benchmark Arena"** tab.
   - Click **"Execute Benchmark"** to see side-by-side performance metrics showing Cognix's token efficiency and multi-hop discovery advantage over traditional Flat RAG and Conversation History Replay!
