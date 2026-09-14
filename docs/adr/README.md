# Architecture Decision Records

Every significant decision, with the alternatives that were rejected and why.
An ADR with no rejected alternative is not a decision, it is a note.

| # | Title | Status | Date |
|---|---|---|---|
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted | 2026-09-11 |
| [0002](0002-orchestration-and-provider-abstraction.md) | Orchestration in application code; provider abstraction over SDKs | Accepted | 2026-09-11 |
| [0003](0003-pgvector-as-vector-store.md) | Postgres + pgvector as the single store | Accepted | 2026-09-11 |
| 0004 | Chunking strategy and hybrid retrieval | _planned, day 4_ | |
| 0005 | Review queue as database state, not a message broker | _planned, day 8_ | |

## Template

```markdown
# NNNN. Title

- **Status:** Accepted | Superseded by NNNN | Deprecated
- **Date:** YYYY-MM-DD
- **Deciders:** Eman Hamam

## Context
## Decision
## Alternatives considered
## Consequences
```
