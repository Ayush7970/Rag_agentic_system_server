# Research Analysis Dashboard — Backend

Backend services for document ingestion, indexing, retrieval, and chat orchestration. Exposes APIs used by the frontend to manage projects, conversations, knowledge base documents, and grounded responses with sources.


---

## System Architecture (Lucid)

Lucidchart (system + data flow):  
https://lucid.app/lucidchart/8caa218d-8dd2-4dc0-8c86-2fb64fbac522/edit

Recommended: export and commit to `docs/diagrams/architecture.png`

![Architecture](architecture1.png)

---

## YouTube Demo

> Replace `YOUR_VIDEO_ID` with your Loom/YouTube video id.

- Demo video: https://www.youtube.com/watch?v=YOUR_VIDEO_ID

[![Demo Video](https://img.youtube.com/vi/YOUR_VIDEO_ID/0.jpg)](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)

---

## What This Backend Does

- **Projects & Conversations**
  - Store project metadata
  - Persist chat threads and messages
- **Knowledge Base**
  - Accept document uploads
  - Chunk + embed documents
  - Store vectors and metadata for retrieval
- **Retrieval**
  - Vector / hybrid search strategies
  - Multi-query retrieval support (depending on configuration)
  - Returns **sources** with page references for UI citation
- **Chat Orchestration**
  - Builds final prompt context from retrieved chunks
  - Produces answers that can include citations and supporting excerpts

---

## Tech Stack (Typical)

- **Python API server** (commonly FastAPI)
- **Poetry** (dependency management)
- **Vector storage** (commonly Postgres + pgvector, or a managed vector DB)
- **LLM provider** (configurable)
- **Background jobs** (optional; for ingestion/indexing at scale)
- **Auth integration** (via frontend JWT/session; backend verifies requests as needed)

> The exact providers depend on environment variables and deployment configuration.

---

## Setup

### 1) Install dependencies

```bash
cd backend
poetry install
```

### 2) Environment Variables

Create `backend/.env`:

```bash
# Server
APP_ENV=local
PORT=8000
CORS_ORIGINS=http://localhost:5173,https://ayush7970.vercel.app

# Database / Vector store
DATABASE_URL=postgresql+psycopg://user:password@localhost:5432/openslate

# Embeddings + LLM
EMBEDDING_MODEL=text-embedding-3-large
LLM_PROVIDER=openai
OPENAI_API_KEY=your_key_here

# Optional: object storage for uploaded docs
STORAGE_PROVIDER=local
STORAGE_PATH=./storage

# Optional: auth verification (if enforcing on backend)
CLERK_JWKS_URL=your_jwks_url_here
CLERK_ISSUER=your_issuer_here
```

---

## Run Locally

### Option A — Run API directly

```bash
poetry run uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

API:
- http://localhost:8000

### Option B — Docker Compose (recommended if you use Postgres/Redis)

```bash
docker compose up --build
```

---

## API Responsibilities (High Level)

Typical endpoints include:

- `POST /projects` / `GET /projects` / `GET /projects/{id}`
- `POST /projects/{id}/chats` / `GET /projects/{id}/chats`
- `POST /projects/{id}/documents` (upload & ingest)
- `GET /projects/{id}/documents` (list)
- `GET /projects/{id}/settings` + `PATCH /projects/{id}/settings`
- `POST /projects/{id}/chat` (ask a question, return answer + sources)

> Keep the frontend and backend aligned on the response shape for:
> - message text
> - sources list (doc name, page, chunk text, score)
> - any streaming tokens/events if supported

---

## Document Ingestion Flow

1. Upload document (PDF/DOCX/PPT/MD/TXT)
2. Extract text
3. Chunk text into passages
4. Generate embeddings for chunks
5. Store:
   - chunk text
   - embedding vector
   - metadata (doc id, page, offsets)
6. Retrieval returns top-k chunks + metadata to power the **Sources** UI

---

## Search Strategies

Common modes supported by the platform UI:

- **Vector Search**: semantic similarity only
- **Hybrid Search**: semantic + keyword (BM25-like) signals
- **Multi-Query Vector**: rewrite question into multiple sub-queries, then merge results
- **Multi-Query Hybrid**: multi-query + hybrid merge

The frontend exposes tunables like:
- **Chunks per Search**
- **Final Context Size**

---

## Folder Structure (Example)

```txt
backend/
  app/
    main.py
    api/
    core/
    services/
    db/
    schemas/
  migrations/
  tests/
  docs/
    diagrams/
  pyproject.toml
  docker-compose.yml
```

---

## Troubleshooting

**CORS errors**
- Ensure `CORS_ORIGINS` includes the frontend URL(s)

**Uploads succeed but no sources appear**
- Ingestion may still be running
- Confirm vector store tables exist and are writable
- Verify embedding model key/config

**Slow retrieval**
- Reduce chunk size or increase overlap tuning
- Add caching
- Ensure vector indexes exist (pgvector index, etc.)

---

## Security Notes

- Validate auth tokens if the deployment requires it
- Apply file-type validation and size limits for uploads
- Store secrets only in environment variables (never commit keys)
