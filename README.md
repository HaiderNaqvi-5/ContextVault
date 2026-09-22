# RAG-cyberify

> A lightweight, grounded knowledge assistant: index local documents, retrieve the relevant passages, and answer with that context.

A Retrieval-Augmented Generation (RAG) application for answering questions from local knowledge documents. It ingests text into a PostgreSQL+pgvector-backed index, retrieves the most relevant chunks, and passes the context to an LLM for grounded answers.

## Features

- Document ingestion from raw text or uploaded text files
- Chunking and embedding workflow for semantic retrieval
- PostgreSQL vector search with pgvector-compatible storage
- Question answering grounded in retrieved context only
- Simple frontend served from the `static/` folder
- FastAPI API for ingestion, retrieval, and health checks

## How it works

```text
Source text → chunking → OpenAI embeddings → PostgreSQL + pgvector index
Question → relevant chunks → OpenAI chat model → context-grounded answer
```

## Tech stack

- Python 3.11+
- FastAPI
- PostgreSQL + pgvector
- psycopg
- OpenAI embeddings and chat models
- HTML/CSS frontend

## Project structure

```text
RAG-cyberify/
├── app/                  # app logic (config, DB, ingestion, retrieval, LLM)
├── db/                   # schema and database assets
├── seed/                 # sample knowledge documents
├── static/               # frontend assets
├── tests/                # automated tests
├── .env                  # local secrets (not committed)
├── .gitignore            # git exclusions
├── requirements.txt      # Python dependencies
├── README.md             # project overview and setup guide
└── .env.example          # example environment file
```

## Prerequisites

- Python 3.11+
- PostgreSQL database with the pgvector extension enabled
- OpenAI API key

## Setup

Clone the repository and enter it:

```bash
git clone https://github.com/HaiderNaqvi-5/RAG-cyberify.git
cd RAG-cyberify
```

Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate
```

On Windows PowerShell, activate it with:

```powershell
.venv\Scripts\Activate.ps1
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Copy `.env.example` to `.env` and fill in the required values:

```env
OPENAI_API_KEY=your_openai_api_key_here
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cyberify_rag
DB_USER=cyberify
DB_PASSWORD=cyberify123
EMBEDDING_MODEL=text-embedding-3-small
EMBEDDING_DIM=1536
CHAT_MODEL=gpt-4o-mini
TOP_K=4
MIN_SCORE=0.20
CHUNK_CHARS=700
CHUNK_OVERLAP=120
```

Make sure the PostgreSQL database exists and the `documents` and `chunks` tables can be created when the app initializes.

## Run the app

```bash
uvicorn app.main:api --reload
```

Then open:

- Frontend: http://127.0.0.1:8000/
- API docs: http://127.0.0.1:8000/docs

The frontend is served by FastAPI, so no separate frontend development server is required.

## Core API endpoints

```http
GET  /api/health
POST /api/ingest
POST /api/ingest/file
GET  /api/documents
DELETE /api/documents/{document_id}
POST /api/ask
```

`/api/ingest/file` accepts UTF-8 `.txt` or `.md` files. Use `/api/ingest` when sending content directly in JSON.

### Example: ingest a document

```bash
curl -X POST http://127.0.0.1:8000/api/ingest \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Internship FAQ",
    "source": "seed/internship-faq.md",
    "text": "The internship is remote and open to students."
  }'
```

### Example: ask a question

```bash
curl -X POST http://127.0.0.1:8000/api/ask \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What types of internship roles are available?",
    "top_k": 4
  }'
```

## Testing

```bash
pytest -q
```

## Notes

- The app answers only from retrieved context.
- It is designed as a grounded document Q&A system, not a general-purpose open-ended chatbot.
- Frontend files are served from the `static/` directory.
- Keep secrets in `.env`; commit only the placeholder values in `.env.example`.
- Retrieval quality depends on the source material, chunking settings, and score threshold configured in `.env`.
