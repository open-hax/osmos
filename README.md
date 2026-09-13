# Osmos

**Osmos** is the ingestion service for the Knoxx/Foresight constellation. It discovers, extracts, and indexes content from configured sources into PostgreSQL, OpenPlanner, and Ragussy.

Extracted from `open-hax/knoxx/ingestion` by [Operation Eta Mu Breakdown](../../docs/migrations/eta-mu-breakdown/).

## Quick start

```bash
# Prerequisites: Java 21+, PostgreSQL, (optional) Redis, Ragussy, Proxx, OpenPlanner

# Set required env vars
export DATABASE_URL="jdbc:postgresql://localhost:5432/futuresight_kms?user=kms&password=kms"
export CONTRACTS_DIR="/path/to/contracts"  # or place contracts/ in cwd

# Run tests (no live services needed)
clojure -M:test

# Run integration tests (needs live OpenPlanner)
clojure -M:integration

# Build uberjar
clojure -M:uberjar

# Run
java -cp target/kms-ingestion.jar clojure.main -m kms-ingestion.server
```

## Docker

```bash
# Build
docker build -t osmos .

# Run (requires external PostgreSQL and contracts mount)
docker run -p 3003:3003 \
  -e DATABASE_URL="jdbc:postgresql://host:5432/futuresight_kms?user=kms&password=kms" \
  -e CONTRACTS_DIR="/app/contracts" \
  -v /path/to/contracts:/app/contracts:ro \
  osmos
```

## Contract discovery

Osmos discovers ingestion sources from EDN contracts:

1. `*contracts-dir*` dynamic var (test injection)
2. `CONTRACTS_DIR` environment variable
3. `<cwd>/contracts` directory

Each tenant has `contracts/<tenant>/sources/*.edn` files. Global defaults live at `contracts/_defaults.edn`.

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `PORT` | `3003` | HTTP server port |
| `DATABASE_URL` | `jdbc:postgresql://localhost:5432/futuresight_kms?user=kms&password=kms` | PostgreSQL connection |
| `REDIS_URL` | `redis://localhost:6379` | Redis connection |
| `CONTRACTS_DIR` | (auto) | Contracts root directory |
| `OPENPLANNER_BASE_URL` | (required for sink) | OpenPlanner API |
| `OPENPLANNER_API_KEY` | (required for sink) | OpenPlanner auth |
| `KNOXX_BACKEND_URL` | `http://knoxx-backend:8000` | Knoxx API (audio/translation) |
| `PROXX_BASE_URL` | (optional) | LLM proxy |
| `RAGUSSY_BASE_URL` | `http://localhost:8000` | Vector search |
| `QDRANT_URL` | `http://localhost:6333` | Qdrant vector DB |
| `TRANSLATION_AGENT_ENABLED` | `false` | Enable translation worker |
| `AUDIO_INDEXING_ENABLED` | `true` | Enable audio indexing |

See `src/kms_ingestion/config.clj` for the full list (47 variables).

## Drivers

| Driver | Purpose | Key config |
|---|---|---|
| `:local` | Local filesystem | `:root-path` |
| `:github` | GitHub repos/issues/PRs | `:repo`, `:token` |
| `:opencode-sessions` | OpenCode session history | `:base-url` |
| `:eta-mu-sessions` | Eta-mu session JSONL | `:root-path` |
| `:audio` | Audio + AI description | `:root-path`, `:chat-url` |
| `:image` | Image + AI labels | `:root-path`, `:chat-url` |
| `:scraper` | Web audio scraper | `:url` |
| `:promptdb` | Structured EDN epistemic | `:root-path` |

## Health check

```
GET /health → 200 OK
```

## License

GPL-3.0-or-later
