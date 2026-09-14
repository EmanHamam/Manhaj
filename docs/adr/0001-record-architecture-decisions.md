# 0001. Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-09-11

## Context

This repository is assessed partly on whether its decisions can be explained and defended,
and it is intended to be usable as teaching material. Decisions made under a 40-hour time
budget are exactly the ones most likely to be forgotten, misremembered, or later mistaken
for accidents.

## Decision

Every significant decision is recorded as a numbered ADR in `docs/adr/`, in Michael
Nygard's format, extended with a mandatory **Alternatives considered** section naming at
least two rejected options and the specific cost that ruled each out.

ADRs are immutable once accepted. A changed decision is a new ADR that supersedes the old
one; the old one stays in the repository with its status updated.

## Alternatives considered

### A decisions section inside the System Design Document — rejected

Decisions would be edited in place as thinking changed, losing the record of what was
believed at the time. The reasoning behind a *reversed* decision is often the most
instructive part, and in-place editing destroys it.

### No formal record; rely on commit messages and PR descriptions — rejected

Commit messages explain a change; they do not survive as a browsable index of why the
system is shaped as it is. A reader with fifteen minutes cannot reconstruct an architecture
from a commit log.

## Consequences

**Positive.** Reviewers and trainees can trace any structural property of the system to a
stated reason. Rejected options are recorded, so "why not X" has an answer.

**Negative.** Roughly 20 minutes per ADR, spent during the tightest part of the schedule.

**Revisit when.** Never — this one is foundational.
