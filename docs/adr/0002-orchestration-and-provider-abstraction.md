# 0002. Orchestration in application code; provider abstraction over SDKs

- **Status:** Accepted
- **Date:** 2026-09-11

## Context

The system runs a multi-agent workflow (Standards Mapper → Curriculum Designer → Item
Generator, with a deterministic validation pass and an orchestrator). FR-5 requires a
**named, justified orchestration pattern** with mandatory controls: a max-iteration
breaker, per-step timeout, retry with backoff, and graceful degradation to plain RAG. FR-4
requires agents to communicate through **typed contracts, not free-form text**, each with a
restricted tool set and a defined termination condition.

Separately, §4 forbids the domain and application layers from depending on any LLM SDK,
and sets an acceptance test: swapping the LLM provider, embedding model or vector store
must require configuration plus one adapter.

The obvious move is to reach for an orchestration product — n8n, Langflow, or LangGraph.
Two decisions are therefore taken together here, because they are the same decision seen
from two sides: **where orchestration logic lives**, and **how model providers are reached**.

## Decision

**Orchestration is a supervisor + state machine written in C# inside `Manhaj.Application`.**
Agents are classes implementing `IAgent`, exchanging C# records. The supervisor owns a
`RunState`, a step budget, per-step `CancellationTokenSource` timeouts, Polly retry with
exponential backoff and jitter, and an explicit degradation transition to plain RAG. Every
transition emits a domain event, which serves as both the SSE progress stream (FR-6) and
the persisted trace (FR-9).

**Model access goes through a single `ILlmProvider` port** covering completion, streaming,
tool calling and embeddings, with two adapters — Gemini (hosted free tier) and Ollama
(local) — implemented with raw `HttpClient` in `Manhaj.Infrastructure`, selected by an
ordered fallback chain in configuration.

## Alternatives considered

### n8n — rejected

Orchestration logic would live in an external runtime's JSON graph: outside the solution,
outside version control's meaningful diff, outside the test suite. The FR-5 controls become
things gestured at rather than owned — there is no way to unit-test that a max-iteration
breaker trips when the breaker is a node's retry setting. Typed contracts between agents
(FR-4) degrade to JSON blobs passing through a canvas. It also adds a service to
`docker compose` whose state is not covered by the repository's migrations.

### Langflow — rejected

Same structural objection as n8n. Additionally, the orchestration would be defined in a
tool that a reader cannot inspect from the repository alone, which conflicts directly with
the requirement that any run be inspectable step-by-step and that the build be explainable
line by line.

### LangGraph — rejected

The closest fit conceptually: it is a real state-machine library and it takes the
orchestration problem seriously. Rejected on two counts. First, it is Python, which forks
the stack for the single most architecturally central component and puts orchestration
behind a process boundary from the domain model it orchestrates. Second, it would own the
state machine that FR-5 asks me to design and justify — importing the answer to the
question being asked.

### Semantic Kernel as the provider layer — rejected

Would work, and would be faster to write. Rejected because it invites SDK types into
application-layer signatures (kernel, function, plan abstractions), which is precisely the
coupling §4's acceptance test is designed to detect. Raw `HttpClient` against two documented
REST APIs is roughly 200 lines per adapter and makes the "no SDK above Infrastructure"
claim trivially true rather than carefully argued.

### An autonomous planner agent instead of a fixed supervisor — rejected

More impressive in a demo, but the termination conditions become emergent rather than
defined, which FR-4 explicitly requires them not to be. In a domain whose stated risk is
plausible-but-wrong output, a non-deterministic control flow is the wrong instinct.

## Consequences

**Positive.** All five FR-5 controls are ordinary testable code — four unit tests with a
stubbed `ILlmProvider` cover breaker, timeout, retry and degradation. Agent contracts are
compile-time checked. The full run is reconstructable because the trace is emitted by the
same events that drive the UI. Swapping providers is one configuration line, demonstrable
in the video. Nothing needed to run the system requires a paid tier.

**Negative.** Roughly 400–600 lines of supervisor and adapter code written by hand that a
tool would have supplied. No visual canvas for a non-engineer to edit the workflow.

**Addressing the visual gap.** The Workflow Studio in the UI renders the run as an animated
DAG driven by the real step events, which covers the demonstration need without moving
orchestration out of the application layer. It also renders something a canvas cannot:
actual token counts, costs and retrieved chunk IDs per step.

**Revisit when.** Non-engineers need to author workflows themselves, or the number of
distinct workflows exceeds roughly five. Neither applies at MVP scope.
