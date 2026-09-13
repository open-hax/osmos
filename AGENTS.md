# Osmos — Agent Guidance

Ingestion service for the Knoxx/Foresight constellation. JVM Clojure service that discovers, extracts, and indexes content from configured sources.

## Quick commands

```bash
# Tests (no live services needed)
clojure -M:test

# Integration tests (needs live OpenPlanner)
clojure -M:integration

# Build uberjar
clojure -M:uberjar

# Run
java -cp target/kms-ingestion.jar clojure.main -m kms-ingestion.server

# Docker
docker build -t osmos .
docker run -p 3003:3003 -e DATABASE_URL="..." -e CONTRACTS_DIR="/app/contracts" -v /path/to/contracts:/app/contracts:ro osmos
```

## Architecture

- **server.clj** — Main entry, startup sequence (DB → contracts → workers → scheduler → watcher → HTTP)
- **contracts/** — EDN contract loader and resolver
- **drivers/** — Pluggable source drivers (local, github, audio, image, scraper, etc.)
- **jobs/** — Job execution, backpressure, ingestion support
- **api/** — Reitit routes, SSE event bus, query/workspace support
- **translation/** — Translation worker (optional, Knoxx-dependent)

## Contract discovery

1. `CONTRACTS_DIR` env var
2. `<cwd>/contracts` directory

Each tenant: `contracts/<tenant>/sources/*.edn`. Defaults: `contracts/_defaults.edn`.

## Runtime services

Required: PostgreSQL. Optional: Redis, Ragussy, Proxx, OpenPlanner, Knoxx backend, Qdrant.

## License

GPL-3.0-or-later
