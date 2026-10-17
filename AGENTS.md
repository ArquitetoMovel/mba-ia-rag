# Repository Guidelines

## Project Structure & Module Organization

- `app/` contains the Python application. `ingest.py`, `index.py`, and `manifest.py` prepare and maintain the vector index; `query_planner.py`, `retrieve.py`, and `rerank.py` implement retrieval stages.
- `app/rag_pipeline.py` owns the shared RAG workflow. Keep `rag_chat.py` and `api.py` focused on terminal and HTTP presentation.
- `knowledge_base/` contains Portuguese Markdown documents with YAML metadata. Generated chunks and the index manifest live in ignored `data/`.
- `test.http` provides manual API requests. No automated test directory exists.

## Build, Test, and Development Commands

Use Python 3.11+ and run commands from the repository root:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
docker compose up -d
```

Populate `.env` before running the application. Compose starts Postgres with pgvector.

- `python -m app.main`: validate configuration, database connectivity, and pgvector.
- `python -m app.ingest`: validate documents and write `data/chunks.jsonl`.
- `python -m app.index`: incrementally index new, changed, or removed documents.
- `python -m app.rag_chat --debug`: run interactive RAG with diagnostics.
- `python -m app.retrieve "Qual é o SLA para P1?"`: inspect retrieval independently.
- `uvicorn app.api:app --reload`: start the local API.

There is no separate build step. Indexing and RAG queries require configured OpenAI access.

## Coding Style & Naming Conventions

Follow existing Python style: four-space indentation, double-quoted strings, type hints, `snake_case` functions/modules, `PascalCase` classes, and `UPPER_SNAKE_CASE` constants. Use Pydantic models for structured requests and results. Group imports by standard library, third-party packages, and local modules. No formatter or linter is configured.

## Testing Guidelines

No automated framework or coverage threshold is configured. Run relevant smoke checks above and execute `test.http` through VS Code REST Client. Verify supported answers, debug output, reranking disabled, clarification, missing knowledge, and empty-input rejection. For indexing changes, rerun indexing to confirm unchanged documents are skipped. Document validation results in the PR.

For direct database validation, open the container console with `docker compose exec postgres sh`, then run:

```bash
psql -U postgres -d rag_db -c "{QUERY}"
```

Replace `{QUERY}` with SQL, for example `SELECT COUNT(*) FROM fcai_knowledge_base;` after indexing.

## Commit & Pull Request Guidelines

Follow the observed Conventional Commits style: `feat: add …` or `chore: ignore …`. Keep commits focused. PRs should describe the behavior change, link relevant issues, list validation performed, and include sample requests/responses for API changes. Update `README.md` when workflows change.

## Configuration & Knowledge-Base Rules

Never commit `.env`, credentials, or generated `data/`. Preserve application-controlled tenant/product/status filters and application-validated sources. Keep ingestion/indexing outside chat runtime. New documents must include every field in `app/ingest.py::REQUIRED_METADATA`; new document types or plans also require updating `DocType`, `Plan`, and the planner prompt catalog before ingestion/indexing.
