# ADR 0005 — Three-step abuse ladder with temporary suspension

- **Status:** Accepted
- **Date:** 2026-10-14

## Context

Some interactions are not legitimate order attempts: harassment, profanity, spam,
or repeated nonsense. The Bot must defend itself without punishing a genuine
customer who is just confused or repeating a question.

## Decision

Classify **Abuse in code, not by the Agent alone**, and respond with a fixed
ladder:

1. **Friendly redirect** — the Bot politely steers back to ordering.
2. **Manager notified + 15-minute Suspension** — the Bot pauses replying to that
   phone number for 15 minutes.
3. **Owner notified + Handoff** — the Bot stops replying for the conversation.

A **Loop** (the customer repeating the same conversational state 3 times with no
progress) is treated as Abuse and enters the same ladder.

Suspension state is **ephemeral** (in memory); it is not persisted.

## Consequences

- **Good:** deterministic, auditable, and independent of model mood; a real
  customer is only briefly paused before escalation.
- **Good:** both Manager and Owner are informed before a Handoff.
- **Cost:** in-memory suspension resets if the server restarts. Acceptable at this
  scale; a restart is an operator-visible event anyway.
