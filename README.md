
# Manhaj — Domain Copilot for Curriculum & Assessment Design

> ITI Technical Instructor (Post-Graduate Training) — Technical Assessment
> **Variant: D3T5** — Education domain · Human review queue twist

An agentic RAG platform that ingests a corpus of occupational standards, competency
frameworks and item-writing guidance, answers questions with verifiable citations, and
runs a multi-agent workflow that turns a target role into a module outline and a set of
assessment items — with a lead instructor holding the pen through a full review queue.

Three binding principles govern the build:

1. **Grounded, never guessing.** Every claim traces to a source chunk. *"Not enough
   information in the corpus"* is a correct and required answer.
2. **The human holds the pen.** No assessment item is published without explicit
   approval, enforced in the domain layer rather than the UI.
3. **Everything is observable.** Any run is reconstructable afterwards: which agent ran,
   which tools, which chunks, what it cost.

---

## Assigned variant and its derivation

- Domain =   **D3 — Education: curriculum & assessment design**
- Twist  =   **T5 — Human review queue**


**D3 requires:** Standards Mapper · Curriculum Designer · Item Generator (+ orchestrator);
lead instructor approves items; guards against plausible-but-wrong items via an automated
validation pass.

**T5 requires:** assignment, priority, SLA timers, escalation, approve / reject /
edit-with-comment, and reviewer statistics.

---

## Status

🚧 **In progress.** This section is updated as increments land; see
[the project board](../../projects) for what is planned, in flight, and deliberately
deferred.

| Area | Status |
|---|---|
| Repository, CI | ✅ |
| Solution skeleton, provider abstraction | ⏳ |
| Ingestion pipeline | ⏳ |
| Hybrid retrieval + citations | ⏳ |
| Evaluation harness | ⏳ |
| Agents + orchestration | ⏳ |
| Review queue (T5) | ⏳ |
| Documentation & teaching pack | ⏳ |

---

## Quick start

_Assumes Docker and nothing else. Full instructions land with the first runnable
increment; the commands below are the target contract._

```bash
git clone https://github.com/EmanHamam/Manhaj.git
cd manhaj
cp .env.example .env          # then set GEMINI_API_KEY, or use the local profile below
docker compose up             # api, web, postgres+pgvector
```

Running with **no API key at all** (fully local, via Ollama):

```bash
docker compose --profile local up
```

Free API key: <https://aistudio.google.com/apikey> (Google AI Studio, free tier).
Provider abstraction is mandatory in this build precisely so that a free tier running
out is a configuration change, not a rewrite — see
[ADR-0002](docs/adr/0002-orchestration-and-provider-abstraction.md).

### Configuration

Every environment variable is documented inline in [`.env.example`](.env.example).

### Tests and the evaluation harness

```bash
dotnet test backend/Manhaj.sln     # unit, integration (Testcontainers), contract
dotnet run --project eval          # golden set + adversarial cases, prints metrics
```

### Seeded demo accounts

| Role | Email | Can do |
|---|---|---|
| Curriculum Designer | `designer@manhaj.local` | ingest, ask, run workflows, submit for review |
| Lead Instructor | `instructor@manhaj.local` | all of the above + claim, approve/reject/edit, escalations, stats, settings |

Passwords are in `.env.example` and apply to local development only.

### 5-Minute Demo Path

_Numbered script landing with the first end-to-end increment._

---

## Architecture

Clean Architecture. The domain and application layers depend on **no** LLM SDK, vector
store SDK or web framework — a constraint enforced by an automated architecture test
rather than asserted in prose, so that swapping the LLM provider, embedding model or
vector store is configuration plus one adapter.

```
backend/src/
  Manhaj.Domain/          entities, value objects, domain errors    (no dependencies)
  Manhaj.Application/     use cases, ports, agents, supervisor      (-> Domain only)
  Manhaj.Infrastructure/  EF Core, pgvector, providers, prompts     (-> Application)
  Manhaj.Api/             minimal APIs, SSE, auth, OpenAPI, DI       (composition root)
```

Full detail: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) ·
[`docs/SYSTEM-DESIGN.md`](docs/SYSTEM-DESIGN.md) · [ADRs](docs/adr/)

---

## Documentation

| Document | Purpose |
|---|---|
| [`docs/BRD.md`](docs/BRD.md) | Business requirements, personas, traceability matrix |
| [`docs/SYSTEM-DESIGN.md`](docs/SYSTEM-DESIGN.md) | Target architecture (Part A) vs implemented MVP + gap table (Part B) |
| [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) | C4 L1–3, sequence, data-flow, ER, layer diagrams, ADRs |
| [`docs/SECURITY.md`](docs/SECURITY.md) | OWASP Web + LLM Top 10 controls mapped to threats |
| [`docs/EVALUATION.md`](docs/EVALUATION.md) | Golden set results, real numbers, failure analysis |
| [`docs/AGENTIC-WORKFLOW.md`](docs/AGENTIC-WORKFLOW.md) | How AI was governed as a participant in this build |
| [`docs/AI-USAGE-LOG.md`](docs/AI-USAGE-LOG.md) | What was delegated, what was written by hand, where the AI was wrong |
| [`teaching/`](teaching/) | Slides, lab sheet, outcomes map, common trainee mistakes |

---

## Videos

| | Link |
|---|---|
| Product demo (5–8 min) | _pending_ |
| Teaching sample (10 min) | _pending_ |

---

## Declarations

- **No starter or template repository was used.** This repository begins from an empty
  GitHub repository; every file is authored for this submission.
- **All corpus data is synthetic.** No real personal data appears anywhere in the
  repository or its history.
- **AI tools were used** throughout, under the governance described in
  [`docs/AGENTIC-WORKFLOW.md`](docs/AGENTIC-WORKFLOW.md), with an honest record of
  failures in [`docs/AI-USAGE-LOG.md`](docs/AI-USAGE-LOG.md).

## License

[MIT](LICENSE)
