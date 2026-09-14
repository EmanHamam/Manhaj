# 0003. Postgres + pgvector as the single store

- **Status:** Accepted
- **Date:** 2026-09-11

## Context

§4 requires a relational store **and** a vector store, both under migrations. FR-2 requires
**hybrid retrieval** — dense plus keyword — with a documented fusion method, and metadata
filtering as a candidate enhancement. The corpus is version-sensitive: occupational
standards supersede one another, so retrieval must be able to filter by framework version
and effective date.

Packaging requires that `docker compose up` brings up the whole system on a clean machine,
and the submission must be verifiable by a reader who has Docker and fifteen minutes.

## Decision

A single Postgres instance (`pgvector/pgvector:pg17`) serves as both the relational store
and the vector store. Chunk embeddings are a `vector(768)` column with an HNSW index;
keyword search uses a `tsvector` column with a GIN index. Both are queried in one SQL
round-trip and fused in application code.

Each chunk row persists its `embedding_model` and `dimensions` alongside the vector, so a
model change cannot silently mix incompatible vectors in one index.

## Alternatives considered

### MongoDB Atlas Vector Search — rejected

Would require running Postgres *and* Atlas: two stores, two authentication paths, two
migration stories, and a second authorization surface for ownership checks. More decisively,
it is cloud-only. A grader on a train, or a paused free-tier cluster, turns
`docker compose up` into a broken quick start — and the brief explicitly asks that the
system be verifiable on a clean machine. Hybrid retrieval would also span two systems,
turning a single fused query into cross-store coordination.

### Qdrant — rejected

A genuinely good vector database, and it runs locally in Docker. Rejected because it adds a
second service and a second migration mechanism to satisfy a requirement one service already
meets, and because metadata filtering on version and effective date sits more naturally
beside the relational rows that already carry those fields. The cost of the extra service is
real and the benefit at this corpus size (roughly 150 pages) is not.

### Chroma — rejected

Convenient for prototyping, weakest of the options on migrations and on the operational
story the System Design Document has to tell. Choosing it would make Part A's target
architecture read as a repudiation of Part B rather than an extension of it.

### Separate Postgres for relational and a dedicated vector service — rejected

Splits a small system for no benefit at MVP scale while doubling the local footprint.

## Consequences

**Positive.** One service, one connection string, one migration history. Hybrid retrieval is
a single query with both arms and a `WHERE` clause for version filtering. Ownership checks
are the same mechanism used everywhere else. Works fully offline, which also serves the
"no paid tier required" constraint. Everything a reviewer needs is inside one container.

**Negative.** pgvector's HNSW is slower than a purpose-built vector database at large scale,
and index build time grows with corpus size. At roughly 150 pages this is not observable.
Postgres full-text ranking is not true BM25; the fusion method compensates by using rank
positions rather than raw scores.

**Revisit when.** The corpus passes roughly 10⁶ chunks, or p95 retrieval latency exceeds
200 ms. Recorded in the SDD Part A as a managed vector database in the target architecture,
with the migration path and cost noted in the gap table.
